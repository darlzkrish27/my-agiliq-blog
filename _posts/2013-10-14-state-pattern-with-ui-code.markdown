---
layout: post
title:  "State pattern with UI Code"
date:   2013-10-14 16:39:27+05:30
tags: design pattern
author: akshar
---
This blog tells how I used state pattern in my UI code, and explains state pattern with JavaScript and jQuery.

Disclaimer:

* This blog will not talk much about benefits of State pattern as there is already a lot written about it. It only gives a practical example of using state pattern with UI code.
* I am not much confident about difference between State pattern and Strategy pattern. Let's discuss if I misunderstood State pattern.

###Use case:
* I have a page with two input fields.
* Page initially has one title field and one body field.
* There is a '+Add' button under title and body:
<img src="http://agiliq.com/dumps/images/20131014/initial.png" style="width: 443px;" alt="State pattern with javascript"/>
* The page can be in two states which are the two radio buttons at top i.e 'Add one' and 'Add both'.
* Functionality of '+Add' differs based on what state(radio button) is selected.

###Different scenarios:
Our assumption at start of each scenarios is that page has only one title and one body.

####'Add one' is selected and user clicks on '+Add' of title
The page starts looking like:
<img src="http://agiliq.com/dumps/images/20131014/add-title.png" style="width: 443px;" alt="State pattern with javascript"/>

If he clicks on '+Add' of title again, it starts looking like:
<img src="http://agiliq.com/dumps/images/20131014/add-two-title.png" style="width: 443px;" alt="State pattern with javascript"/>

####'Add one' is selected and user clicks on '+Add' of body
The page starts looking like:
<img src="http://agiliq.com/dumps/images/20131014/add-body.png" style="width: 443px;" alt="State pattern with javascript"/>

####'Add both' is selected and user clicks on '+Add' of title
Page starts looking like:
<img src="http://agiliq.com/dumps/images/20131014/locked-add-title.png" style="width: 443px;" alt="State pattern with javascript"/>

In 'Add both' both title and body field should be added irrespective of which '+Add' button is clicked.

If the user clicks any of the '+Add' button again, one more combination of title and body will be added. So, it will start looking like:
<img src="http://agiliq.com/dumps/images/20131014/locked-add-twice.png" style="width: 443px;" alt="State pattern with javascript"/>

So, you got the idea of how '+Add' buttons will behave depending on which radio button is selected at that instant. User can switch between the different radio buttons and '+Add' will behave differently.

###Let's deal with code now.

Code for initial page.

    <html>
        <head>
            <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
        </head>
        <body>
            <div style="width:100%;">
                <div style="float:left; width:50%">
                    <input type="radio" name="state" checked="checked" value="one">Add one</input>
                </div>
                <div style="float:left; width:50%">
                    <input type="radio" name="state" value="both">Add both</input>
                </div>
            </div>

            <div class="input_div">
                <input type="text" class="title" placeholder="Title" />
                <br/>
                <a href="" class="add">+Add</a>
            </div>

            <br/>
            <br/>
            <br/>

            <div class="input_div">
                <input type="text" class="body" placeholder="Body" />
                <br/>
                <a href="" class="add">+Add</a>
            </div>
        </body>
    </html>

Let's add the javascript for '+Add' buttons.

    <script type="text/javascript">
        $(document).ready(function () {
            $("a.add").click(function (e) {
                e.preventDefault();
                var state = $("input[name='state']:checked").val();
                if (state=="one") {
                    var lastInput = $(this).prev().prev();
                    var newInput = lastInput.clone();
                    $(this).before(newInput);
                    $(this).before("<br/>");
                }
                else if(state=="both"){
                    var inputDivs = $(".input_div");
                    inputDivs.each(function (index, inputDiv) {
                        var $inputDiv = $(inputDiv);
                        var addButtton = $inputDiv.find("a.add");
                        var lastInput = $inputDiv.find("input:last");
                        var newInput = lastInput.clone();
                        addButtton.before(newInput);
                        addButtton.before("<br/>");
                    });
                }
            });
        });
    </script>

Add this script in `<head>` of the page.

Switch between two radio buttons at top and notice the difference in behaviour of '+Add' buttons.

Two states required two conditions in the '+Add' button handler. The number of conditions in handler will increase as number of state increases.

Also, we only concerned ourselves with '+Add' field functionality. There can be a use case with '-Delete' field functionality. So handler for '-Delete' will have as many conditions as number of states too. So the delete handler would look like:

    $(".some_class_on_delete_buttons").click(function () {
        if (state=="one") {
            //do something
        }
        else if (state="both") {
            //do something else
        }
        //And in case there are more state
        //there will be more conditionals
    });

With more states and more functionalities this approach becomes unmaintainable.

I want two things:

* I want to get rid of conditionals.
* I want to keep all the functionality for a particular state at a single place. eg: For state 'Add one', I want both add and delete in a single class. For state 'Add both', I want add and delete in a separate class.

It needs to be accomplished in following way:

* I will keep separate classes for different states.
* All the state specific classes need to extend from a super class.
* Super class will define the abstract methods and subclasses should provide the actual implementation for those methods.

In case you are in hurry, see the final code [at github](https://gist.github.com/akshar-raaj/6973560).

Superclass **State** looks like:

    var State = function () {};

    State.prototype.add = function (addButton) {
        throw new Error("It needs to be implemented by subclasses");
    };

Subclasses **AddOneState** and **AddBothState** correspond to states 'Add One' and 'Add Both':

    var AddOneState = function () {};
    AddOneState.prototype = Object.create(State.prototype);
    AddOneState.prototype.add = function (addButton) {
        var lastInput = addButton.prev().prev();
        var newInput = lastInput.clone();
        addButton.before(newInput);
        addButton.before("<br/>");
    };

    var AddBothState = function () {};
    AddBothState.prototype = Object.create(State.prototype);
    AddBothState.prototype.add = function (addButton) {
        var inputDivs = $(".input_div");
        inputDivs.each(function (index, inputDiv) {
            var $inputDiv = $(inputDiv);
            var addButtton = $inputDiv.find("a.add");
            var lastInput = $inputDiv.find("input:last");
            var newInput = lastInput.clone();
            addButtton.before(newInput);
            addButtton.before("<br/>");
        });
    };

Notice that I just moved the conditional statements to **add()** of proper class.

I need another class. Let's call it **Page**. This class will represent the web page we are looking at. This class will have an instance variable called **state** that keeps track of current state of Page i.e whether 'Add One' or 'Add Both' is selected.

    var Page = function (state) {
        this.state = state;
    };
    Page.prototype.add = function (addButton) {
        this.state.add(addButton);
    };

The above three lines are the core of state pattern. Page has an **add** which delegates to **add** of current state.

Initially the page has 'Add One' selected, so we need to keep the initial state as an instance of **AddOneState**.

    var page = new Page(new AddOneState());

And finally, let's add the handlers:

    $("a.add").click(function (e) {
        e.preventDefault();
        page.add($(this));
    });

    $("input[name='state']").change(function () {
        var stateVal = $("input[name='state']:checked").val();
        if (stateVal == "one") {
            page.state = new AddOneState();
        }
        else {
            page.state = new AddBothState();
        }
    });

So, final script and html looks like:

    <html>
        <head>
            <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
            <script type="text/javascript">
                $(document).ready(function () {
                    var State = function () {};

                    State.prototype.add = function (addButton) {
                        throw new Error("It needs to be implemented by subclasses");
                    };

                    var AddOneState = function () {};
                    AddOneState.prototype = Object.create(State.prototype);
                    AddOneState.prototype.add = function (addButton) {
                        var lastInput = addButton.prev().prev();
                        var newInput = lastInput.clone();
                        addButton.before(newInput);
                        addButton.before("<br/>");
                    };

                    var AddBothState = function () {};
                    AddBothState.prototype = Object.create(State.prototype);
                    AddBothState.prototype.add = function (addButton) {
                        var inputDivs = $(".input_div");
                        inputDivs.each(function (index, inputDiv) {
                            var $inputDiv = $(inputDiv);
                            var addButtton = $inputDiv.find("a.add");
                            var lastInput = $inputDiv.find("input:last");
                            var newInput = lastInput.clone();
                            addButtton.before(newInput);
                            addButtton.before("<br/>");
                        });
                    };

                    var Page = function (state) {
                        this.state = state;
                    };
                    Page.prototype.add = function (addButton) {
                        this.state.add(addButton);
                    };

                    var page = new Page(new AddOneState());

                    $("a.add").click(function (e) {
                        e.preventDefault();
                        page.add($(this));
                    });

                    $("input[name='state']").change(function () {
                        var stateVal = $("input[name='state']:checked").val();
                        if (stateVal == "one") {
                            page.state = new AddOneState();
                        }
                        else {
                            page.state = new AddBothState();
                        }
                    });
                });
            </script>
        </head>
        <body>
            <div style="width:100%;">
                <div style="float:left; width:50%">
                    <input type="radio" name="state" checked="checked" value="one">Add one</input>
                </div>
                <div style="float:left; width:50%">
                    <input type="radio" name="state" value="both">Add locked combination</input>
                </div>
            </div>

            <div class="input_div">
                <input type="text" class="title" placeholder="Title" />
                <br/>
                <a href="" class="add">+Add</a>
            </div>

            <br/>
            <br/>
            <br/>

            <div class="input_div">
                <input type="text" class="body" placeholder="Body" />
                <br/>
                <a href="" class="add">+Add</a>
            </div>
        </body>
    </html>


Though the script becomes huge compared to the initial script which contained conditionals, it is much easier to maintian and scale.

Say we need to handle one more state, we will create another class corresponding to that state and write an appropriate **add()** method on that class. But we wouldn't need any change in **Page** to accomodate a new state and our script will keep working fine.

Say we want to add delete functionality, we can add a **delete()** method on all the states and define a **delete()** on Page. **delete** of Page will delegate to **delete** of current state.

You should read more about State Pattern to make sense of this example.

References:

* I came across State pattern while reading "Refactoring" by Martin Fowler.

