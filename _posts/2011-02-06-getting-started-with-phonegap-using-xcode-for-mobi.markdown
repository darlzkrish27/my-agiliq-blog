---
layout: post
title:  "Getting started with PhoneGap using Xcode for Mobile app development"
date:   2011-02-06 21:48:33+05:30
tags: android
author: shabda
---
This is next in the series of apps I am building using various mobile technologies.
It is the same app as build using Java for Android for calculating the Tax payable (in India). You can 
get the code for [Objective-C](https://github.com/agiliq/TaxCalculatorIndia), [Java-Android](https://github.com/agiliq/TaxCalculatorAndroid), and [PhoneGap](https://github.com/agiliq/TaxCalculatorPhoneGap).


First the impressions
------------------------

[Phonegap](http://www.phonegap.com/) was the easiest to work with among the Objective-C, Java and PhoneGap, by far.
I created the app as easily as

Step 1. Write the app and test it in Browser using the Chrome developer tools. 
Step 2. Start a project in Xcode.
Step 3. Copy the HTML and CSS files to project directory and export to simulator.

The experience was very good, (After spending time with Obj-C and Java), as I could use a Chrome console as a repl,
something which I sorely missed.
	
The exported app does *look* native, with controls which look coming from browser. But with some CSS you can go
a long way here.


Getting started
-------------------------

1. Get the Xcode and the Iphone SDK.
2. Download PhoneGap and install from PhoneGap iOS, this will create a new user template for PhoneGap in Xcode.
3. Create a PhoneGap project in Xcode.
4. Edit the www/index.html

Code like it is the web
----------------------------
I don't like working with the code->compile->check in simulator cycle. You can work much faster if you work with the browser.

I edited the index.html, adding the Jquery library for easy DOM manipulation, and a bit of CSS to align.

Lay out the Layout
---------------------------

You can layout your UI using the common HTML elements. Here is a snippet from the finished app.

	 <div class="input">
	 	<label>
	 		Your investments under 80D are:
	 	</label>
	 	<input type="number"  placeholder=0 id="infra_investment"></input>
	</div>


	 <div class="input">
	 	<label>
	 		Your housing loan interests paid are:
	 	</label>
	 	<input type="number"  placeholder=0 id="housing_interest"></input>
	</div>
	
These are standard HTML5 elements. Note that we were able to use HTML5 only elements like

	type="number"  placeholder=0
	
which set the keyboard type and placeholder which you can do via `android:inputType` or editing in Interface Builder.

Code the code
---------------------
We will write the code to get the input values and then calculate the tax. We are using Jquery functions
for UI manipulation. Here is what the code looks like.


			$("#calculate_tax").click(function(){
            income = $("#income").val();
        	investment = $("#investment").val();
			infra_investment = $("#infra_investment").val();
			housing_interest = $("#housing_interest").val();
			medical_premium =  $("#medical_premium").val();
			console.log(""+income+investment+infra_investment+housing_interest+medical_premium);
			if (!income){
				income = 0;
			}
			.....
				var tax = calculateTax(income, investment, infra_investment, housing_interest, medical_premium);
				console.log(""+tax);			
				calculated_tax  = $("#calculated_tax");
				calculated_tax.html(tax);
	        });

			function getTax(taxableIncome){
			.....
			
[Full listing here](https://github.com/agiliq/TaxCalculatorPhoneGap/blob/master/www/index.html)

Put on iPhone 
--------------------
Once you like what you see in chrome, you can put this code on simulator. Build and run from Xcode and you are done.

Resources
-------------------

1. [Code on github](https://github.com/agiliq/TaxCalculatorPhoneGap)
2. [Similar app using Java(For Android)](https://github.com/agiliq/TaxCalculatorAndroid)
3. [Similar app using Objective-C](https://github.com/agiliq/TaxCalculatorIndia)
4. [Our starting Android development tutorial](http://agiliq.com/blog/2011/02/starting-android-app-developement-from-zero-to-app/)


