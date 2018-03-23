---
layout: post
title:	"Dissecting Phonegap\'s architecture"
date:   2012-09-06 00:00:00
author:   thejaswi
tags:   ios

[Apache Cordova](http://incubator.apache.org/cordova/) is a open source
cross-platform framework for building native mobile applications using
HTML, CSS and JavaScript. It started off as Phonegap, a project of
Nitobi Software before it was acquired by Adobe Systems. The code for
the platform was donated to the Apache Software foundation and is
currently being incubated as \"Apache Cordova\".

[Phonegap](http://phonegap.com/) is now a distribution of Apache Cordova
(analogous to Ubuntu being a Linux distribution) brought to you by
Adobe. Since Apache Cordova is licensed under the permissive Apache
Software License, Adobe Phonegap may technically be integrated with
proprietary software (though there\'s no evidence for the same yet).

This post is not going to discuss how to build a cross-platform mobile
app using Phonegap and if you are here expecting that, you are better
off checking their
[docs](http://docs.phonegap.com/en/2.0.0/guide_getting-started_index.md.html).
In this post, we are going to see how Phonegap apps work ie how the
javascript component is able to communicate with the native APIs and
vice-versa.

The Cordova guys have taken a lot of pain keep a consistent JS interface
on the client side but underneath there is a large divergence between
each platform.

We are going to discuss the architectures of android and iOS since these
are the most widely used platforms and restrict ourselves to version 2.0
of Cordova.

Every phonegap app has the following components:

-   A chrome-less browser. On iOS and Android, it is WebKit (UIWebView
    on iOS and WebView on android to be specific).
-   JS to Native bridge to allow for communication between the HTML
    application and the native platform.
-   A native to JS bridge to allow the native platform to talk to the
    HTML application.

[![Android Phonegap architecture](http://agiliq.com/dumps/images/20120906/android_phonegap.png){.align-center
width="470px"
height="332px"}](http://agiliq.com/dumps/images/20120906/android_phonegap.png)

In android, by default the JS to Native Bridge is set to Prompt (yes,
you saw it right, the venerable JS prompt dialog box). The JS functions
(like camera, contacts etc) are converted to Prompt commands by the
cordova javascript and intercepted by the WebView
[onJsPrompt](http://developer.android.com/reference/android/webkit/WebChromeClient.html#onJsPrompt(android.webkit.WebView,%20java.lang.String,%20java.lang.String,%20java.lang.String,%20android.webkit.JsPromptResult))
and based on a specific signature calls the respective native plugin
(camera, contacts etc).

It is also possible to change the JS to Native bridge and another way of
communication is through the JS\_Object bridge. When the WebView is
loaded and the JS bridge is set, the WebView adds a [Javascript
Interface](http://developer.android.com/reference/android/webkit/WebView.html#addJavascriptInterface(java.lang.Object,%20java.lang.String))
which calls a Java object (calling the respective native plugin based on
the arguments of the interface).

There is yet another JS to Native bridge (currently experimental) which
calls the native plugins by triggering changes in the location URL.

Now we come to the other end of the communication ie the Native to JS
bridge. By default, the bridge is set to polling and the javascript
keeps polling the native side for a response every 50 milliseconds. This
is very suboptimal but the solution works on the largest number of
devices and in most setups.

Just like the JS to Native bridge, the Native to JS bridge can also be
changed. Another Native to JS bridge is the XHR bridge (called the
HANGING\_GET internally as a reference to a long lived XHR connection).
This bridge runs a callback server locally and responds to the XHR
requests.

There is yet another bridge that uses Java internal reflection on the
webview to call the methods but is available only on Android 3.2+. There
are a couple of other bridges but don\'t seem to be in use at this point
in time.

Now let\'s look at how the iOS phonegap apps work.

[![iOS Phonegap architecture](http://agiliq.com/dumps/images/20120906/ios_phonegap.png){.align-center
width="470px"
height="332px"}](http://agiliq.com/dumps/images/20120906/ios_phonegap.png)

Compared to Android, iOS has fewer bridges. On iOS 4.2 and below, the JS
and Native bridge communicate with each other through an iframe. The JS
calls are stored in a JS queue which is read and executed by the native
component.

The other bridge is an XHR bridge which makes calls to a fake URL with
the commands in the header. These commands are intercepted, serialized
and then executed.

For the Native to JS bridge, the iOS phonegap apps have only one bridge
and all the communication happens through a UIWebView method called
[stringByEvaluatingJavaScriptFromString](https://developer.apple.com/library/ios/#documentation/UIKit/Reference/UIWebView_Class/Reference/Reference.html).

Now that we know how Phonegap apps work, we can write better apps and
plugins and guess where we can improve our application\'s performance.

ml
