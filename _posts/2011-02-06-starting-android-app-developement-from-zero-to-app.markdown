---
layout: post
title:  "Starting Android app developement: From zero to app"
date:   2011-02-06 17:30:21+05:30
tags: java
author: shabda
---
We recently started with Mobile Application development. I am learning Android using the [Commonsware book](http://Commonsware.com/), and highly recommend it.
This is a very short guide to getting you running your first Android app.

What will we build
------------------------
We will build a simple tax calculator for India, per the rules [given here](http://en.wikipedia.org/wiki/Income_tax_in_India) for tax calculation. In this app, we need to gets users
income and various tax deductible expenses. After that we need to update the UI to show the tax. You can see [the final app](https://market.android.com/details?id=com.agiliq.taxcalc)
and [code on github](https://github.com/agiliq/TaxCalculatorAndroid) here.


Setting up
---------------
1. Install Java and Eclipse 
2. Install Android SDK and add ons.
3. Instal ADT for eclipse.
4. Create a Android project from within Eclipse.

What the project contains:
-----------------------------
1. AndroidManifest.xml: This is the entry point for the app. It decides the properties including which Java file will start the App.
2. YourApp.java: This is a java class which extends `Activity` and has the `onCreate` method, which decides where to get the Layout from.
3. main.xml: By default `YourApp.java` will look for Layout in the files res/layout/main.java
4. strings.xml: While not necessary you will have your strings in an external xml file here.

There are a lot of other files, but these are the ones you are going to need for our App.

How does the Layout go to the Java file
-------------------------------------------
You will need to modify the Layout in your Java code, but you defined your layout in main.xml. How does Java get it?
Eclipse auto-generates a file called R.java, which will get your layout and all declared fields in Java. R.java, *should not*
be edited directly, it is automatically generated. (As is everything in the gen folder.)

Lets write the layout.
--------------------------------
You can build the Layout in a GUI tool provided by Eclipse, but I found it *very* underpowered. I preferred to write it directly in XML.

There are a lot of widgets available, but for this app we are going to need three.

1. A Uneditable text field. Its called a TextView.
2. I Editable text view. Its called a EditText
3. A button. Its called a Button.
4. Make it scrollable. Its called a ScrollView

Layout your app in the preferred order. I added 5 EditText to get the income and tax deductible values. We will read these values and display them in
a TextView. We need a button to know when to calculate the tax.

You will notice that instead of setting the text values via normal strings we are doing it via `android:text="@string/income_tax_init"`. This references
values from the Strings.xml file which looks like this.

You can set a lot of values on your UI elements for setting things like color. The import one is `android:id`. You must set a value here if you want to make
the widget available in your Java code.

[Your code will look like this after you are done.](https://github.com/agiliq/TaxCalculatorAndroid/blob/master/res/layout/main.xml)

Let write the Java code
-----------------------------------
In our Java code we ned to do,

1. Get the Input value for the fields.
2. Calculate tax using these values.
3. Update the UI to show the tax.

All of this needs to be handled after a click, so your public class needs to `implements Button.OnClickListener`
and implement the `onClick`.

You can get the fields you declared in XML this way:

	(EditText)findViewById(R.id.income);
	
And the values from them as,

	income.getText().toString();
	
Now we can get the values and parse them as `int` via `Integer.parseInt`, and calculate tax via normal Java semantics.

Once we have the tax, the UI can be updated via:

	tax.setText(""+tax);
	
	
[Your final Java code will look like this.](https://github.com/agiliq/TaxCalculatorAndroid/blob/master/src/com/agiliq/taxcalc/TaxCalculator.java) 

Bridging XML and Java
----------------------------
Most of the Java<->XML bridging is done via R.java, by making the XML widgets available in Java. You need to set the function to be
called on button click.

This is done via the `android:onCLick` property on the `Button`, in the `main.xml`.

And we are done
---------------------
You can set the logo and package it for sale in the Android marketplace now. :)


Resources
--------------

1. [Code for this app](https://github.com/agiliq/TaxCalculatorAndroid)
2. [This app on Android marketplace](https://market.android.com/details?id=com.agiliq.taxcalc)
3. [Similar code for Iphone and Objective C](https://github.com/agiliq/TaxCalculatorIndia)
4. [Commonsware: The book I am reading](http://commonsware.com/)




