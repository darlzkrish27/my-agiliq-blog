---
layout: post
title:  "Getting started with Titanium development for Android and Iphone"
date:   2011-02-08 11:54:11+05:30
tags: iphone
author: shabda
---
This is the third post in our mobile app development series. ( You may want to read [Phonegap](http://agiliq.com/blog/2011/02/getting-started-with-phonegap-using-xcode-for-mobi/) and [Android with Java](http://agiliq.com/blog/2011/02/starting-android-app-developement-from-zero-to-app/).)

Like last time we will build an App which allows calculating the tax payable, [per the rules here](http://en.wikipedia.org/wiki/Income_tax_in_India). We will
use [Titanium Mobile](http://www.appcelerator.com/products/titanium-mobile-application-development/)

Installing Titanium
------------------------
[Download and install](http://www.appcelerator.com/products/download/) the fairly small Titanium from their sites. Once you download and start it, 
it will download and install more components. You can then create a new project from their UI.

Layout the Layout
---------------------
You can add UI widgets and lay them out using Javascript. You need to use Titanium's API, to create
the UI. 

Titanium execution starts in app.js. We will modify this file to add the layout code.

You can add a label like this

	Titanium.UI.createLabel({
		color:'#999',
		text:label_text,
		font:{fontSize:20,fontFamily:'Helvetica Neue'},
		textAlign:'left',
		width: 'auto',
		height: 'auto',
		top: 5
	});
	


Similarly you can create a Button and TextInput as

	Titanium.UI.createTextField(
	{
	....
	}
	);
	
	Titanium.UI.createButton({
		...[Options]
	})
	
Create a window as `Titanium.UI.createWindow({})`. and add these Items to it via `window.add`. Your widgets have been added to the UI, and should show up, in the simulator. [Here is what mine looks like](http://skitch.com/shabda/rpue1/ios-simulator). [You can get the code for the layout at here](https://github.com/agiliq/TaxCalculatorTitanium/blob/master/Resources/app.js). 

Code the code
---------------------
We now need to add the code to handle button clicks, calculate the tax, and update the UI with the tax. Since our code for this is going to be 
Javascript, we can easily reuse the tax calculation parts from our Phonegap app.


	
	function getTax(taxableIncome){
	    var taxOnThisSlab;
	    if (taxableIncome < 160000) {
	        return 0;
	    }
	 ///More code to calculate the tax
	//....

	}


	function calculateTax(income, investment, infra_investment, housing_interest, medical_premium){
		investment = Math.max(0, Math.min(investment, 100000));
		 ///More code to calculate the tax
		//....
		
	}
	
Now we add a `EventListener` to the Button we added and this is used to calculate the tax and update the Label for showing the text.

	calculateTaxBut.addEventListener("click", function(){
		var income = income_f.value;
		var investment = investment_f.value;
		//More code to get the values and update the tax area.
		//.....
		var tax = calculateTax(income, investment, infra_investment, housing_interest, medical_premium);
		calculated_tax.text = ""+tax;
	
    });


And we are done
-------------------
Overall, the experience with Titanium was **OK**. The Titanium developer UI could have been better, the UI 
is Unintuitive. Building for Android is very slow. Lunch button in their developer UI seems to stop working at random intervals.
However the promise of building native apps is very alluring, and I would be working more with Titanium.

Resources
-------------
1. [Finished code on Github](https://github.com/agiliq/TaxCalculatorTitanium)
2. [Tutorial: Building the same app with Obj-c] - Todo
3. [Tutorial: Building the same app with Phonegap](http://agiliq.com/blog/2011/02/getting-started-with-phonegap-using-xcode-for-mobi/)
4. [Tutorial: Building the same app with Java:Android](http://agiliq.com/blog/2011/02/starting-android-app-developement-from-zero-to-app/)
5. [Code: Phonegap app on Github](https://github.com/agiliq/TaxCalculatorPhoneGap)
6. [Code: Java app on Github](https://github.com/agiliq/TaxCalculatorAndroid)
7. [Code: Obj-C app on Github](https://github.com/agiliq/TaxCalculatorIndia)



