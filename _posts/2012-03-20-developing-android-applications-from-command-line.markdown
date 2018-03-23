---
layout: post
title: Developing android applications from command line
date:   2012-03-20 01:00:00
author:   thejaswi
tags:   terminal
---

Here at Agiliq, we also develop cross-platform HTML5 mobile
applications. Using Eclipse to create an android project (one time task)
and edit html and javascript files for an android app is an overkill.
Wouldn\'t it be great, if you could use your favourite text editor to
edit html and js files and then fall back to the terminal to deploy the
android app? We are going to see exactly this in the post.

First, let\'s install the android SDK starter package(I assume that you
have installed JDK and ant). Head over
[here](http://developer.android.com/sdk/index.html) and install the SDK
for your platform. Since I use ubuntu, I download the Linux platform SDK
to my home directory. Uncompress the download and set the path to load
the executables:

    $ tar xvf android-sdk_r16-linux.tgz
    $ export PATH=$PATH:~/android-sdk-linux/tools:~/reqs/android-sdk-linux/platform-tools
    $ android sdk

[![Android SDK manager](http://agiliq.com/static/dumps/images/20120320/android_sdk_manager.png){.align-center
width="90.0%"}](http://agiliq.com/static/dumps/images/20120320/android_sdk_manager.png)

If you don\'t wish to keep exporting the PATH variable everytime, you
can set it in your \~/.bashrc or \~/.bash\_profile file.

The android sdk command opens the Android SDK manager. Download the
requisite SDK packages for the android versions that you plan to target.
I plan to develop the app for Gingerbread and Icecream Sandwich phones
and hence download the Android 4.0.3 (API 15) and Android 2.3.3 (API 10)
components.

Let\'s check which versions of the SDK are installed from the terminal:

    $ android list targets

While developing the app, we need a way to test the app and hence let\'s
create Android Virtual Devices (AVD) that will create virtual devices
that we\'ll run later through the emulator:

    $ android avd

[![Android AVD manager](http://agiliq.com/static/dumps/images/20120320/android_avd_manager.png){.align-center
width="90.0%"}](http://agiliq.com/static/dumps/images/20120320/android_avd_manager.png)

Create as many AVDs as the android SDK versions that you downloaded in
the previous step.

To view a list of AVDs we just created on the terminal, use the
following command:

    $ android list avd

Let\'s test if the AVDs we created run on the emulator. I want to run
the AVD named Gingerbread on the emulator:

    $ emulator @Gingerbread

[![Android Emulator](http://agiliq.com/static/dumps/images/20120320/android_emulator.png){.align-center
width="90.0%"}](http://agiliq.com/static/dumps/images/20120320/android_emulator.png)

We\'ve setup the basic infrastructure for being able to develop android
apps.

Creating an android project
---------------------------

Let\'s create an android project (for creating a phonegap based android
project, skip to the next section):

    $ android create project -n TestAndroidProj -t 'android-15' -p ~/android_proj -k com.example -a TestProjActivity

The value (TestAndroidProj) to the -n switch is the name of the project,
the value (android-15) to the -t switch is the android SDK version the
app targets. The value to the -p switch mentions the path of the android
project. The -k switch requires a valid (java) package name and the -a
switch takes the name of the initial
[Activity](http://developer.android.com/reference/android/app/Activity.html).

After running the above command, an android project should have got
successfully created. Check if everything\'s as expected:

    $ cd android_proj/
    $ ls src/com/example/TestProjActivity.java 
    src/com/example/TestProjActivity.java

**(BONUS)** Installing and creating a Phonegap app
--------------------------------------------------

If you don\'t plan to create an HTML5 app or don\'t plan to use
Phonegap, you can safely skip this section. See you later\...

Immediately after installing the android SDK, clone Cordova (formerly
Phonegap) from this
[location](git://git.apache.org/incubator-cordova-android.git) (or clone
it from a [github](https://github.com/apache/incubator-cordova-android/)
tag for a stable version). Add the bin directory under the cloned
directory to your \`PATH\`:

    $ git clone git://git.apache.org/incubator-cordova-android.git
    $ export PATH=$PATH:~/incubator-cordova-android/bin

Let\'s create a Cordova project, which is a wrapper around
`android create project` but additionally sets up phonegap dependencies
(the jar and the js files) for you:

    $ cd ~/incubator-cordova-android
    $ ./bin/create ~/phonegap_android_proj com.example TestProjActivity 3

The first argument is the path of the project, the second is the (java)
package name, the third is the name of the initial activity that loads
your index.html and the fourth argument is the android sdk id (taken
from `android list targets`, in my case \'android-15\' id is \'3\').

Let\'s check if the project has been successfully created:

    $ cd ~/phonegap_android_proj/
    $ ls assets/www/
    cordova-1.5.0.js  index.html  main.js  master.css
    $ ls libs/
    cordova-1.5.0.jar
    $ ls res/xml/
    cordova.xml  plugins.xml

The following sections are common to both native android projects and
phonegap based android projects.

Ant commands
------------

Now that we are done with the project and write all the code for the
app, we need to be able to test it in the emulator. So let\'s start the
emulator (mentioned above) and then start the Android Debug Bridge
(adb). It is recommended you start adb as a superuser because if you
later connect your phone, it will have the requisite permissions to
access it:

    $ sudo ~/android-sdk-linux/platform-tools/adb start-server

Check if the adb server process is running before you proceed further:

    $ ps aux|grep adb
    root     12589  0.0  0.0  20184   948 pts/3    Sl   16:43   0:00 adb fork-server server

### Deploying the app to the emulator

In your project root, run the ant commands to deploy the app (signed
with the debug key) to the emulator:

    $ cd ~/android_proj
    $ ant clean debug install

The app should show up on your emulator and you can test it out. If you
plan to use the phone, the same set of commands will work. You might
want to close the emulator then or adb will complain that both the
emulator and the device are connected. To debug and view log messages
use the Dalvik Debug Monitor Server (ddms):

    $ ddms

Once you are satisfied with the app, you will want to sign it and
publish it to the market (Google Play).

### Signing the app to deploy to marketplace

First, let\'s generate a private key that will be used to sign the
application:

    $ keytool -genkey -v -keystore app_signing.keystore -alias release \
      -keyalg RSA -keysize 2048 -validity 10000

The keytool that is part of the JDK is used to create the private key.
The -keystore argument\'s value is the name of the output file where the
keys are stored. The -alias is a human readable name for the key (as
multiple keys may be stored) in the keystore which can be used to refer
the key later on. The encryption algorithm is set to RSA with a keysize
of 2048 bits and a validity of 10000 days. Keep the generated keystore
file very safe as this identifies you on the Google Play store.

After you have successfully generated your private key, let\'s compile
the app in the release mode:

    $ ant clean release

You will notice under the bin directory of your project, a file of the
format \<project\_name\>-release-unsigned.apk (in our case,
\'TestAndroidProj-release-unsigned.apk\'). After we are done with this,
we have to sign the app with the private key we created previously:

    $ jarsigner -keystore app_signing.keystore -digestalg SHA1 -sigalg MD5withRSA bin/TestAndroidProj-release-unsigned.apk release

The jarsigner utility uses the keystore (created previously) while
specifying the digest algorithm, signature algorithm, the release
android application and the name of the alias to be used from the
keystore.

Let\'s verify if everything went fine:

    $ jarsigner -verify bin/TestAndroidProj-release-unsigned.apk

You should get a \'jar verified\' message and might want to repeat the
signing process if you didn\'t.

Finally, let us align the generated application file (apk) before
submitting to Google Play store.:

    $ zipalign -v 4 bin/TestAndroidProj-release-unsigned.apk bin/TestAndroidProj.apk

\'4\' specifies that the files in the apk should be aligned to the
4-byte boundary. The next argument is the input signed application file
and the last argument is the output file that is used to upload to the
Google Play store.

The only command that we\'ll use frequently from the terminal is
`ant clean debug install` and by doing so we can avoid running Eclipse
(which hogs memory like there\'s no tomorrow).

**Bonus**: If you are an Emacs user, you might want to use
[android-mode](http://marmalade-repo.org/packages/android-mode) which
has key bindings for most of the above mentioned commands and other
goodies.
