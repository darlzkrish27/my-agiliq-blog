---
layout: post
title:  "Django-SocialAuth - Login via twitter, facebook, openid, yahoo, google using a single app."
date:   2009-08-27 14:50:33+05:30
tags: yahoo
author: shabda
---
TL;DR version: Here is an app to allow logging in via twitter, facebook, openid, yahoo, google, which should work transparently with Django authentication system. (@login_required, User and other infrastructure work as expected.) [Demo](http://socialauth.uswaretech.net/) and [Code](http://github.com/uswaretech/Django-Socialauth/tree/master).Longer version follow:

-----------------------
We are releasing our new app. [**Django-Socialauth**](http://github.com/uswaretech/Django-Socialauth/tree/master). This app makes it awfully easy,
to allow users to login your site using Yahoo/Google/Twitter/Facebook/Openid. A demo is available [here](http://socialauth.uswaretech.net/).

This is released under an [Attribution Assurance License](http://www.opensource.org/licenses/attribution.php). A copy of the same is
provided included with the code.

After installing this app, you can use @login_required on any view and users identified
via any means can access protected content.

--------------------------------
We provide services to integrate and implement this, for a low price of USD 1600.
Please contact us at [licenses@uswaretech.com](mailto:licenses@uswaretech.com) to discuss your exact needs.

---------------------------

The README is copied here for convenience.

### What it does.

Allow logging in via various providers.

#### Logging In



This is a application to enable authentication via various third party sites.
In particular it allows logging in via

1. Twitter
2. Gmail
3. Facebook
4. Yahoo(Essentially openid)
4. OpenId

Libs you need to install

1. [python-openid](http://pypi.python.org/pypi/python-openid/) (`easy_install`)
2. python-yadis (`easy_install`)
3. [python-oauth](http://oauth.googlecode.com/svn/code/python/oauth/ )(`easy_install`)


The API Keys are available from

* [http://www.facebook.com/developers/createapp.php](http://www.facebook.com/developers/createapp.php)
* [https://developer.yahoo.com/dashboard/createKey.html](https://developer.yahoo.com/dashboard/createKey.html)
* [https://www.google.com/accounts/ManageDomains](https://www.google.com/accounts/ManageDomains)
* [http://twitter.com/oauth_clients](http://twitter.com/oauth_clients)

#### How it works.

Openid: Users need to provide their openid providers. Talk to the providers and
login.  
Yahoo: Yahoo is an openid provider. Talk to Yahoo endpoints. (Endpoint: http://yahoo.com)  
Google: Google is a provider. Talk to them. (Endpoint: https://www.google.com/accounts/o8/id)  
Facebook: Facebook connect provides authentication framework.  
Twitter: We use Twitter Oauth for authentication. In theory, Oauth shouldn't be
used for authentication. (It is an autorisation framework, not an authentication one),
In practice it works pretty well. Once you have an access_token, and a name, essentially
authenticated.  

References

0. [Demo of app](http://socialauth.uswaretech.net/)
0. [Code for app](http://github.com/uswaretech/Django-Socialauth/tree/master)
1. [http://openid.net/developers/](http://openid.net/developers/)
2. [http://developer.yahoo.com/openid/](http://developer.yahoo.com/openid/)
3. [http://code.google.com/apis/accounts/docs/OpenID.html](http://code.google.com/apis/accounts/docs/OpenID.html)
4. [http://apiwiki.twitter.com/OAuth-FAQ](http://apiwiki.twitter.com/OAuth-FAQ)
5. [http://developers.facebook.com/connect.php](http://developers.facebook.com/connect.php)

Below the hoods
-----------------

1. For all providers(except Facebook) there are two urls and views. (start and done)
2. Start sets up the required tokens, and redirects and hands off to the correct
provider.
3. Provider handles authentication on their ends, and hands off to Us, providing
authorization tokens.
4. In done, we check if the user with these details already exists, if yes, we
log them in. Otherwise we create a new user, and log them in.

For all of these, we use standard django authenication system, with custom
`auth_backends`, hence all existing views, and decorators as `login_required`
will work as expected.

Urls
---------
    
    /login/ Login page. Has all the login options  
    /openid_login/ AND /openid_login/done/  
    /yahoo_login/ AND /yahoo_login/done/  
    /gmail_login/ AND /gmail_login/done/  
    /twitter_login/ AND /twitter_login/done/  
    /facebook_login/done/ We dont have a start url here, as the starting tokens are  
    set in a popup, as per FB Connect recommendations.

Implementation
----------------

0. Install required libraries.
1. Get tokens and populate in localsettings.py
2. Set the token callback urls correctly at Twitter and Facebook.
3. Add the OpenId middleware. Set the Authentication backends. (Set in localsettings.example.py)






