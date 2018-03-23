---
layout: post
title:  "Disqus and Disqus SSO"
date:   2014-11-17 20:00:29+05:30
tags: Disqus
author: Rakesh
---
Outsourcing the comments and discussions part to a third party widget always helps us to save a lot of effort. When you feel that if it is okay to share your thoughts and discussions with someone out there who serves the capabilities of social integration and spam detection won't it be a cake walk?

Disqus is such a kind of blog comment hosting service which is platform-agnostic and URL-agnostic. It is built with javascript and backend technology, Python using Django. Let us get to know how to add this to our Django app.

Adding Disqus widget to a site is pretty straight forward and effortless. Here are the steps to follow:

1. Create an account on Disqus and fill the details at https://disqus.com/admin/create/
2. Once you finish registration, access the universal code
3. In the next following page, get the script and use it wherever we want to display comments
4. Update the settings file with DISQUS_SHORTNAME

The first step includes signing up in to Disqus and getting an account. Once we are done with it, we will be heading to a home page where we can find "Add disqus to your site" on top left. If we click it, we will be redirected to site profile page where we should fill the details like the site name and unique Disqus URL which we call it as "disqus_shortname". Once the registration is completed it will show up the general settings page where we can select the platform i.e univeral code, wordpress, blogger, Tumblr, square space and so on. As we are planning to add Disqus to Django app we should select Universal code. The Universal code is a script which looks like this:

    <div id="disqus_thread"></div>
    <script type="text/javascript">
        /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
        var disqus_shortname = 'try1'; // required: replace example with your forum shortname

        /* * * DON'T EDIT BELOW THIS LINE * * */
        (function() {
            var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
            dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
            (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
        })();
    </script>
    <noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>

The script can be used in the template wherever we want our comments section to show up. That delivers us the comments section in our app with one step remaining. Access the "disqus_shortname" i.e. the unique Disqus URL that we specified in the process of registration and add it in the settings file of the Django app.

eg.
    DISQUS_WEBSITE_SHORTNAME = "mydjangoapp"

Disqus SSO
----------------- 

Commenting on a thread requires a person to log in with their social accounts. It would be an extra step for the people to log in again for commenting after logging in to the app. To abate this fatigue of logging in again we can use the "single sign on".

As of now SSO is available to Disqus as a free add on. To add to our app we should request the support team of Disqus through a contact form. Once we send a request to support team it takes like 24 hours to get the access to SSO console.

Steps to register Disqus application

1. Register the application at https://disqus.com/api/applications/register/
2. Once the registration is completed the secret keys and the public keys can be accessed at https://disqus.com/api/applications/
3. Add the keys to settings file as DISQUS_PUBLIC_KEY and DISQUS_SECRET_KEY
4. To avail console, SSO and stuff we can go to https://disqus.com/api/applications/
5. Pass existing user data

The first part to start using it is creating a Remote Domain which can be used as a reference between Disqus and the user accounts. We can do it right away at https://disqus.com/api/sso/. Add the public key and secret key of the application in the settings. We can access the keys at https://disqus.com/api/applications/ under the app name. Use HMAC-SHA1 to pass the user details to the Disqus to authenticate. The message that we pass should contain id, username and email. We can generate the HMAC-SHA1 the following way:

    HMAC->SHA1(secret_key, message + ' ' + timestamp)

Once the message is generated the following script need to be used in the page where we want the Disqus to show up.

    var disqus_config = function () 
    {
        // The generated payload which authenticates users with Disqus
        this.page.remote_auth_s3 = '<message> <hmac> <timestamp>';
        this.page.api_key = 'public_api_key';
    }

In Django we can accomplish this by registering a simple tag with function to generate the HMAC-SHA1 and using that tag in the template where we are displaying the Disqus comments. The flow moves on the following way:

Firstly, we should access the value of the DISQUS_SECRET key from the settings. In the next step we should create a json packet of the data attributes, i.e 'id', 'username', 'email'. Then we should encode the json packet to base64 and we should generate a time stamp for signing the message. To pass the secret key and the user details we should generate the HMAC signature. Thus acquired HMAC signature should be returned as a script tag to insert the SSO message. At the end the function appears as in the following code. Check out the [Disqus SSO Example](https://gist.github.com/krvc/9afc17db9eb8d01b7655)


The template tag should be used in the place where we want the Disqus comments to appear. 

And that ends up the process of adding SSO to Disqus.

