---
layout: post
title:  "Writing an e-mail application with Lamson - II"
date:   2011-04-05 03:14:10
author:   thejaswi
tags:   e-mail
---

In the [last
post](http://agiliq.com/blog/2011/04/writing-an-e-mail-application-with-lamson-i/),
we saw how to create the skeleton of a basic email application using
Lamson. In this part, we\'ll see how to write a handler (in the
controller) to open a ticket in Unfuddle, the project management tool we
use at Agiliq.

If you look at the `config/settings.py` file, you\'ll see a `handlers`
attribute that should be updated to match the file that contains the
rules for mail routing. In this case, let us create a `unfuddle.py`
under the `app/handlers` directory and update the `config/settings.py`:

    handlers = ["app.handlers.unfuddle"]

In the `app/handlers/unfuddle.py`, the handler code would be as below:

    from lamson.routing import route, route_like, stateless
    from config.settings import UNFUDDLE_USERNAME, UNFUDDLE_PASSWORD
    import httplib2, json

    UNFUDDLE_API_ENDPOINT = "https://%(subdomain)s.unfuddle.com/api/v1"

    @route("(subdomain)\+(project)@(host)", subdomain="\w+", project="\w+", host=".+")
    def START(message, subdomain=None, project=None, host=None):
       UNFUDDLE_API_URL = UNFUDDLE_API_ENDPOINT %({"subdomain": subdomain})
       h = httplib2.Http()
       h.add_credentials(UNFUDDLE_USERNAME, UNFUDDLE_PASSWORD)
       _, content = h.request("%(url)s/projects.json" %({"url": UNFUDDLE_API_URL}), method="GET")
       json_content = json.loads(content)
       proj = None
       for proj_iter in json_content:
           if proj_iter["short_name"] == project:
               proj = proj_iter
       if not proj:
           return START
       _, tickets = h.request("%(url)s/projects/%(id)s/ticket_reports/dynamic.json?conditions_string=status-neq-closed"
                              %({"url": UNFUDDLE_API_URL, "id": proj["id"]}))
       ticket_json = json.loads(tickets)
       tickets = []
       if ticket_json["groups"]:
           tickets = ticket_json["groups"][0]["tickets"]
       for ticket in tickets:
           if ticket["summary"] == message.base["Subject"]:
               _, content = h.request("%(url)s/projects/%(id)s/tickets/%(ticket_id)s/comments.json"
                                        %({"url": UNFUDDLE_API_URL, "id": proj["id"], "ticket_id": ticket["id"]}),
                                      method="POST", 
                                      body="""{"comment": {"body": "%(body)s" } }""" 
                                        %{"body": message.body().replace('"','\"')},
                                     headers={"Accept": "application/json", 
                                              "Content-Type": "application/json"})
               return START
       _, content = h.request("%(url)s/projects/%(id)s/tickets.json" %({"url": UNFUDDLE_API_URL, "id": proj["id"]}), 
                              method="POST",
                              body="""{"ticket": {"summary": "", "description": "", "priority": "3"} }"""
                                 %({"summary": message.base["Subject"].replace('"', '\"'), 
                                    "description": message.body().replace('"', '\"') }),
                           headers={"Accept": "application/json", 
                                    "Content-Type": "application/json"})
       return START


    @route_like(START)
    @stateless
    def ERROR(message, subdomain=None, project=None, host=None):
        return START

For a conventional application, e-mail is routed on the basis of rules
mentioned in configuration files (aliases, macros etc) and then dropped
off to a separate process (like fetchmail or procmail) which finally
delivers the e-mail to it\'s destination. In lamson, all this routing is
implemented through \'[State
Machines](http://en.wikipedia.org/wiki/Finite-state_machine)\'. This
makes for cleaner code (unified at one place and process) and easy to
develop and debug. Each function generally represents a state and
returns the next state (could be the same state itself) where the
application should proceed. Lamson uses data stores to store the state.
By default, it uses the `MemoryStorage` but it\'s very easy to write a
custom data storage to say store the mails in a database etc.

In our example app, there are two states to the state machine. They are
\'START\' and \'ERROR\'. The ERROR state is a special state in which the
application finds itself when there\'s any error. The \'START\' state
accepts all mails with the from address in the regexp format
`(?P<subdomain>\w+)\+(?P<project>\w+)@(?P<host>.+)`. These captured
regular expression groups are passed as keyword arguments to the
\'START\' function. Based on these keyword arguments, we use the
[Unfuddle API](http://unfuddle.com/docs/api) (which uses HTTP Basic
Auth) to create a ticket if it doesn\'t exist else attach a comment to
the ticket.

We\'ll have to restart the lamson server to be able to reflect the new
changes. You can then use the local mutt config file to be able to send
mails to the server and try out your changes (for example, send a mail
to <test+abc@localhost>):

    mutt -F muttrc

Keep a watch at the `logs/lamson.log` files while developing because
this information may be invaluable.

In the next part, we\'ll see how to setup and deploy this application.

::: {.note}
::: {.admonition-title}
Note
:::

The code for this e-mail application is available at
<https://github.com/agiliq/unfuddle/tree/master/email2ticket>
:::
