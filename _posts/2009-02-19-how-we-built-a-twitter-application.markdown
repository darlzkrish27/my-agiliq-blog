---
layout: post
title:  "How we built a Twitter Application"
date:   2009-02-19 13:50:52+05:30
categories: web2.0
author: lakshman
---
Twitter, just like many other Web2.0 sites, has an excellent API. So much so, writing a twitter app is considered to be "Hello World" of Web2.0 Programming.

I wrote an application recently. It is [DM++](http://dmplusplus.com). By default, this application allows you to receive direct messages from twitter users you have @replied even if you currently don't follow them. You can even configure application to be able to receive direct messages from people you have @replied in only last 2 days, or receive Dm from all other twitter users.

Look how many people are currently complaining [Cant DM U as U dont follow me](http://search.twitter.com/search?q=%22cant+dm%22) or read our [Faq](http://faq.dmplusplus.com)

Now, let me illustrate some steps in building this application. 

One of the modules is where I need to find out whether the userx follows usery. This request needs authentication headers added for twitter to respond, else U receive a "Not authorized" response


    import urllib2,base64,simplejson
    
    def is_follows(follower, following):
        
        theurl = 'http://twitter.com/friendships/exists.json?user_a=%s&user_b=%s'%(follower, following)
        
        username = 'uname'
        password = 'pwd'
        
        handle = urllib2.Request(theurl)
        
        authheader =  "Basic %s" % base64.encodestring('%s:%s' % (username, password))
        
        handle.add_header("Authorization", authheader)
        
        try:
            return simplejson.load(urllib2.urlopen(handle))
        except IOError, e:
            # This is reached when allocated API requests to IP are completed.
            print "parsing the is_follows json from twitter, failed"
        return False
        
    
    if __name__=='__main__':
        print is_follows(u'scorpion032',u'scobleizer')


This is an independent module, intended so, as this functionality is best that way, simplest without the bloat of any other functions and it is portable.

There is a very good <a href="http://code.google.com/p/python-twitter/">Python-twitter wrapper</a> by Google, many functions on twitter are now very simple, for anything more than finding out if userx follows usery as above, this can be very useful.


    import twitter
    api=twitter.Api('uname','pwd')


Thats it and you have an authenticated api object from which U can do nearly all twitter functions. Infact it is working with this wrapper, that tempted me to write this application. There are a host of things you can do with this wrapper including what is shown below, that I have used.

This code retrieves all Direct Messages that are received since it was last retrieved. We store the fetchinfo table pk=1 field with the last date time it was retrieved
    
    
    try:
        f=FetchInfo.objects.get(pk=1).last_datetime
    except:
        f=None   
    dm_list=api.GetDirectMessages(since=f)


Following snippet sends the Direct Messages to the users that can receive it, else it is marked filtered.
'canhesend()' is a function where we find verify if the user is able to send direct messages based on his access_level setting
        

	    if canhesend(adm.sent_by.username,user):
                try:
                    api.PostDirectMessage(user,"via @"+adm.sent_by.username+": "+adm.text)
                    api.PostDirectMessage(adm.sent_by.username,"Your Dm has been sent to @"+user)
                    adm.is_sent=True
                    adm.save()
                except HTTPError:
                    adm.is_filter=True
                    adm.save()
            else:
                adm.is_filter=True
                adm.save()


`adm` is the parameter that loops on direct messages.

Among other important things see how we seperate the usernames and text which are sent to us, there is a very nice input from [stackoverflow](http://stackoverflow.com/questions/558105/string-separation-in-required-format-pythonic-way-with-or-w-o-regex/563299#563299), which we incorporated.

We have a text of the format 
`'@abc [@def ..] This part is text'`
which we want seperated to user names and text part. We want retrieved in the following format:
    
    l=['abc','def']
    s='This part is text'

This is how we went about doing it:

    import re
    rx = re.compile("((?:@\w+ +)+)(.*)")
    t='@abc   @def  @xyz Hello this part is text and my email is foo@ba.r'
    a,s = rx.match(t).groups()
    l = re.split('[@ ]+',a)[1:-1]
    print l
    print s


