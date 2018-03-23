---
layout: post
title:  "How to use jsTree"
date:   2011-10-12 16:30:00+05:30
tags: Treeview
author: akshar
---
Using jsTree

##Problem Statement:
To show a TreeView (hierarchical view of information) structure in a Web page.

I came across this problem when i was required to develop a web application which allows its users to store contacts and their related information(name, email, phone). The contacts would go into a folder and any folder can have any number of subfolders and any subfolder can have any number of contacts. Also, the folder structure can be nested as many levels deep as the user wants. So, how do you show the folder structure in a Web Page?

You might encounter a lot of similar situation. 

###For the problem stated above, What we need is a Treeview structure.
1. Display the first level folders(say Level 1 folder). 

2. User expands a Level 1 folder, expand and show all the subfolders within it.

3. User expands one of the subfolder(Level 2 folder), expand and show all the subfolders(Level 3 folder) within this subfolder and so on.   

While searching for the solution to this problem scenario, i came across jsTree.

jsTree is a jquery plugin which allows displaying a tree structure in a web page. In this post we will see how to use it.

###We will be dividing our work in two parts.
1. Illustrating basic use of jsTree. (A simple example)
2. Solving the problem stated above (using Django)

##Basic use of jsTree:

The example used here is not a practical example but as already specified its just for illustration purpose and to get a basic understanding of jsTree.

####Our example:
1. Consider two categories to be shown on the page.
2. These categories can be considered synonymous to top level folders.
3. Lets say the two categories as "Search Engines" and "Networking Sites".
4. Under "Search Engines" we put Google, Bing and Yahoo. "Google", "Bing" and "Yahoo" form second level folder (immediately below top level folder).
5. Under "Networking Sites" we put Facebook and Twitter. So, "Facebook" and "Twitter" form second level folder.
6. Now under Google, we put Gmail, YouTube and Orkut. These form the third level folders.
7. We must also see some actions that can be performed while using jsTree. So, we will take the user to the link for these elements on being selected, provided the link exists.

Lets create an html page which illustrates this example.

I have put this code at github. You can get the entire code at <a href="https://github.com/akshar-raaj/Using-jsTree">https://github.com/akshar-raaj/Using-jsTree</a>

Let's name the html page as try_jstree.html . Initially the page is:

    <html>
        <head>
            <title>Use jsTree</title>
        </head>
        <body>
            <div id="treeViewDiv">
            </div>
        </body>
    </html> 

Since jsTree is a jquery plugin, it requires jquery to be included in the page before we can use jsTree. So, we downloaded jquery.js and put it in the folder containing the html(try_jstree.html) file. Then we need to download jsTree(named jquery.jstree.js) and we put it in the same folder. Some folders (themes, docs etc.) comes while downloading jsTree. Put all these folders in our working folder.

We have our first commit at this point.

Now, include jquery.js and jquery.jstree.js in the file so that we can use them. At this point code looks like:

    <html>
        <head>
            <title>Use jsTree</title>
            <script src="jquery.js">
            </script>
            <script src="jquery.jstree.js">
            </script>
            <script>
                $(document).ready(function(){
                    
                });
            </script>
        </head>
        <body>
            <div id="treeViewDiv">
            </div>
        </body>
    </html>

We have our second commit at this point.(commit 4c7c7233ccb4d0e2e463375af9c579374ee73624)

Next we need to select the div where the tree will be displayed. We use jquery selector for it. On the selected element we call jstree() and need to pass the required values.

Here, we will be using json data to populate our tree. 

####So, the fields needed by jstree() in such case are:
1. json_data - This is an object.
2. plugins - This is a list.

Hence at this point our code looks like:

    <html>
        <head>
            <title>Use jsTree</title>
            <script src="jquery.js">
            </script>
            <script src="jquery.jstree.js">
            </script>
            <script>
                $(document).ready(function(){
                    $("#treeViewDiv").jstree({
                        "json_data" : {
                            
                        },
                        "plugins" : []
                    });
                });
            </script>
        </head>
        <body>
            <div id="treeViewDiv">
            </div>
        </body>
    </html>

We have our third commit at this point.(commit fd47c9784b7510a8cc3de9209054fbcc39fbac9b)

Now, "json_data" requires an object having a field "data". "data" is a list of the objects which we want displayed on the web page. So, "json_data" would look like

    "json_data": {
        "data":[]
    }

Each entry in "data" is the actual object to be displayed and used on the webpage. So, we will put the objects in "data" so that they can be displayed on the page.

So, "json_data" becomes

    "json_data" : {
                            "data":[
                                {
                                    "data" : "Search engines"
                                },
                                {
                                    "data" : "Networking sites"
                                }
                            ]
                  }

Also, for using json_data we need a plugin named "json_data". For displaying the tree as a tree like structure, we need the plugin named "themes".And for taking the user to a new page upon selecting an entry(which we will be illustrating soon), we need a plugin named "ui". So, "plugins" become

    "plugins" : [ "themes", "json_data", "ui" ]

At this point our code is:

    <html>
        <head>
            <title>Use jsTree</title>
            <script src="jquery.js">
            </script>
            <script src="jquery.jstree.js">
            </script>
            <script>
                $(document).ready(function(){
                    $("#treeViewDiv").jstree({
                        "json_data" : {
                            "data":[
                                {
                                    "data" : "Search engines"
                                },
                                {
                                    "data" : "Networking sites"
                                }
                            ]
                        },
                        "plugins" : [ "themes", "json_data", "ui" ]
                    });
                });
            </script>
        </head>
        <body>
            <div id="treeViewDiv">
            </div>
        </body>
    </html>

We have our fourth commit at this point(commit 3e32dfeefe1adbddf3dc88d4a68e93e834996c0e). Check your page now. You should be able to see these entries on the page.

Next, we need to add subelements to the elements. This is done by adding a "children" attribute to the object with which we need to attach the subelement. "children" is an array containing all the childs. We will associate the subelements as described earlier for our example. In the same step we will add subelements(level 3 elements) to the subelements(level 2 elements).

At this point, our code looks like

    <html>
        <head>
            <title>Use jsTree</title>
            <script src="jquery.js">
            </script>
            <script src="jquery.jstree.js">
            </script>
            <script>
                $(document).ready(function(){
                    $("#treeViewDiv").jstree({
                        "json_data" : {
                            "data":[
                                {
                                    "data" : "Search engines",
                                    "children" :[
                                                 {"data":"Yahoo"},
                                                 {"data":"Bing"},
                                                 {"data":"Google", "children":[{"data":"Youtube"},{"data":"Gmail"},{"data":"Orkut"}]}
                                                ]
                                },
                                {
                                    "data" : "Networking sites",
                                    "children" :[
                                        {"data":"Facebook"},
                                        {"data":"Twitter"}
                                    ]
                                }
                            ]
                        },
                        "plugins" : [ "themes", "json_data", "ui" ]
                    });
                });
            </script>
        </head>
        <body>
            <div id="treeViewDiv">
            </div>
        </body>
    </html>

Check you page now.

You should be seeing a Treeview structure in your page. The screenshot for what your page should look like is [here](http://agiliq.com/dumps/images/20121231/jstree.png).
We have our fifth commit at this point(commit 5276b111ee250220a7d422fe86142f16bb45a1cc).

Next we want to add some additional information(fields) about each object, so that we can operate on those fields. The additional information is stored as field "metadata". "metadata" is again an object containing key-value pairs for the additional information. So, the structure of "metadata" would be 

    "metadata":{}

Let's name the information we want to store as "href". Lets store the value for this key as the url(We need it for our example). For object having "data" as "Gmail", its "metadata" becomes

    "metadata":{"href":"www.gmail.com"}

Similarly for all the objects, and our code would look like

    <html>
        <head>
            <title>Use jsTree</title>
            <script src="jquery.js">
            </script>
            <script src="jquery.jstree.js">
            </script>
            <script>
                $(document).ready(function(){
                    $("#treeViewDiv").jstree({
                        "json_data" : {
                            "data":[
                                {
                                    "data" : "Search engines",
                                    "children" :[
                                                 {"data":"Yahoo", "metadata":{"href":"http://www.yahoo.com"}},
                                                 {"data":"Bing", "metadata":{"href":"http://www.bing.com"}},
                                                 {"data":"Google", "children":[{"data":"Youtube", "metadata":{"href":"http://youtube.com"}},{"data":"Gmail", "metadata":{"href":"http://www.gmail.com"}},{"data":"Orkut","metadata":{"href":"http://www.orkut.com"}}], "metadata" : {"href":"http://youtube.com"}}
                                                ]
                                },
                                {
                                    "data" : "Networking sites",
                                    "children" :[
                                        {"data":"Facebook", "metadata":{"href":"http://www.fb.com"}},
                                        {"data":"Twitter", "metadata":{"href":"http://twitter.com"}}
                                    ]
                                }
                            ]
                        },
                        "plugins" : [ "themes", "json_data", "ui" ]
                    });
                });
            </script>
        </head>
        <body>
            <div id="treeViewDiv">
            </div>
        </body>
    </html>

At this point we have our sixth commit(commit 8bad3a8688c1e848fee4cda99d82f2e369d87f92).

Next we need to attach handler to events(Our event would be selection of any element in the Tree structure, and our handler should take us to the "href" for that element). 

We accomplish this using bind(), similar to how we use in jquery script. The event we are concerned with is "select_node.jstree". So, the bind() function would look like:

    bind("select_node.jstree", function(e, data){})

Whenever an element is selected, we want to get its "href" and take the user to that "href". Also, we make sure that we take the user to new page only when "href" is defined for the element, else we show an alert telling that "href" is not defined for the element.

We attach this bind() function with the div containing the Treeview structure. So, our final code looks like

    <html>
        <head>
            <title>Use jsTree</title>
            <script src="jquery.js">
            </script>
            <script src="jquery.jstree.js">
            </script>
            <script>
                $(document).ready(function(){
                    $("#treeViewDiv").jstree({
                        "json_data" : {
                            "data":[
                                {
                                    "data" : "Search engines",
                                    "children" :[
                                                 {"data":"Yahoo", "metadata":{"href":"http://www.yahoo.com"}},
                                                 {"data":"Bing", "metadata":{"href":"http://www.bing.com"}},
                                                 {"data":"Google", "children":[{"data":"Youtube", "metadata":{"href":"http://youtube.com"}},{"data":"Gmail", "metadata":{"href":"http://www.gmail.com"}},{"data":"Orkut","metadata":{"href":"http://www.orkut.com"}}], "metadata" : {"href":"http://youtube.com"}}
                                                ]
                                },
                                {
                                    "data" : "Networking sites",
                                    "children" :[
                                        {"data":"Facebook", "metadata":{"href":"http://www.fb.com"}},
                                        {"data":"Twitter", "metadata":{"href":"http://twitter.com"}}
                                    ]
                                }
                            ]
                        },
                        "plugins" : [ "themes", "json_data", "ui" ]
                    }).bind("select_node.jstree", function(e, data)
                        {
                            if(jQuery.data(data.rslt.obj[0], "href"))
                            {
                                window.location=jQuery.data(data.rslt.obj[0], "href");
                            }
                            else
                            {
                                alert("No href defined for this element");
                            }
                        })
                });
            </script>
        </head>
        <body>
            <div id="treeViewDiv">
            </div>
        </body>
    </html>

This is the final code and the last commit(commit ff886547287eec114d7020ad66718cae83a1ee37).

We will see how to use jsTree with django, and solve our problem in the next post.

