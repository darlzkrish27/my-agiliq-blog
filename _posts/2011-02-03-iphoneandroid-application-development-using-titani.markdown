---
layout: post
title:  "iPhone and Android application development using Titanium"
date:   2011-02-03 20:52:06+05:30
tags: mobile applications
author: dheeru
---
[Titanium Mobile](http://www.appcelerator.com/products/titanium-mobile-application-development/) is [Appcelerator's](http://www.appcelerator.com/) development platform for developing cross-platform native mobile applications. In this article we will be introducing you to installation and developing applications using Titanium.

## Installation 

Follow the steps mentioned [here](http://guides.appcelerator.com/en/getting_started.html#installing_titanium), to install Titanium and the corresponding sdks for your developement platform

## Hello world application

Creating a new application and aplication structure is elaborately described in [appcelerator gettting started guide](http://guides.appcelerator.com/en/getting_started.html#hello_world!)


## Creating Forms

#### Field Label
	var win = Titanium.UI.currentWindow; // get refernce to the current window
	var label = Titanium.UI.createLabel({
		text: 'Label Name',
		height: 50,
		color: '#000000',
		font:{fontSize:14},
		top: 20, // vertical postion of label on the screen w.r.t screen top
		left: 20, // horizontal postion of label on the screen w.r.t screen left
		width: 200 // width of the label
	});
	win.add(label); // add label to the current window

Check more properties of label in [Titanium Label API](http://developer.appcelerator.com/apidoc/mobile/latest/Titanium.UI.Label-object.html)

#### Text Field
	var text_field = Titanium.UI.createTextField({
		color:'#000000',
		height: 50,
		top: 70, // vertical postion of label on the screen w.r.t screen top
		left: 20, // horizontal postion of label on the screen w.r.t screen left
		width: 200, // width of the field
		borderStyle:Titanium.UI.INPUT_BORDERSTYLE_ROUNDED
	});
	win.add(text_field); // add field to the current window


Check more properties of textfield in [Titanium Text Field API](http://developer.appcelerator.com/apidoc/mobile/latest/Titanium.UI.TextField-object.html)

#### Button
	var button = Titanium.UI.createButton({
		title: 'Button',
		top: 160, // vertical postion of label on the screen w.r.t screen top
		width: 200, // width of the button
		height: 50 // width of the button
	});
	win.add(button); // add button to the current window
Check more properties of button in [Titanium Button API](http://developer.appcelerator.com/apidoc/mobile/latest/Titanium.UI.Button-object)
	
#### Submiting Form Data

Form submission has to be handled via 'click' event listner on button

	button.addEventListener('click', function() { 
		var field_value = text_field.value;
	});

Screenshot of how the form with the above code looks
![Screenshot of form](http://media.agiliq.com/images/blog/iphone_screenshot.png "Form Screenshot")

Accessing database and saving data to database is mentioned in the next section.



## Database

#### Creating database

    var db = Titanium.Database.open('db_name');

'open' returns a reference to the opened database. If the database doesn't yet exist, 'open' creates the database.
	

#### Executing SQL queries

    db.execute('CREATE TABLE IF NOT EXISTS table_name (id INTEGER PRIMARY KEY, field1 VARCHAR, field2 VARCHAR);');


#### Inserting data into table

    var f1 = 'v1';
    var f2 = 'v2';
    db.execute('INSERT INTO table_name (f1, f2) VALUES (?, ?)', f1, f2);


Id of last inserted row is the value of 'lastInsertRowId' property on the database
    var last_insert_id = db.lastInsertRowId;


#### Selecting and looping over the data from table

    var rows = db.execute('select * from table_name');
	
    while (rows.isValidRow()) {
        // check below on how to access field values of a row
	    rows.next();
    }
    rows.close();


#### Accessing the field values of a row by index(zero based)

    var id = rows.field(0);
    var v1 = rows.field(1);
    var v2 = rows.field(2);


#### Accessing the field values of a row by name

    var id = rows.fieldByName('id');
    var v1 = rows.fieldByName('field1');
    var v2 = rows.fieldByName('field2');

More about [Database API](http://developer.appcelerator.com/apidoc/mobile/latest/Titanium.Database-module)

* Titanium.Database.install('meditracker');, raised invalid database path error when tried to run the application on device. So I opted to use Titanium.Database.open



#### Code Resources

* [https://github.com/appcelerator/KitchenSink/](https://github.com/appcelerator/KitchenSink/)
* [https://github.com/agiliq/meditracker](https://github.com/agiliq/meditracker), our sample application for tracking medicine timings.
* [http://itunes.apple.com/in/app/track-my-med/id415622449?mt=8#](http://itunes.apple.com/in/app/track-my-med/id415622449?mt=8#) The [github code](https://github.com/agiliq/meditracker) on Apple App store
* [http://market.android.com/details?id=com.agiliq.trackmymed](http://market.android.com/details?id=com.agiliq.trackmymed) same code on Android stors



