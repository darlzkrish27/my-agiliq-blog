---
layout: post
title:  "Building Chrome Extensions"
date:   2015-03-04 11:40:38+05:30
tags: google
author: karambir
---
In this blog post, we will look at how to build a chrome extension that can work with multiple online services. Specifically we will see how to update Trello web pages with relevant content from Google Drive. For a particular Trello board, we will update trello cards with data from Google Drive document.

Google Chrome extensions are nothing but some html, js files linked together that are running behind the scenes. Before starting out, we need to make sure that *Developer mode* is checked in `chrome://extensions/`. 

Chrome extensions work by telling chrome about our html, js files using a Manifest.json. It will have some meta data like name, description, icon images and more important information like permissions our extension needs, files to load in background. A simple Manifest.json can look like this:

    {
        "name": "Trello Updater",
        "version": "0.1",
        "manifest_version" : 2,

        "description": "Update trello with Google Drive",

        "background": {
            "scripts": ["background.js"],
            "persistent": false
        },

        "browser_action": {
            "default_icon": "icon.png",
            "default_title": "Trello Updater"
        },

        "content_scripts": [
            {
                "matches": ["https://trello.com/*"],
                "js": ["lib/jquery.js",
                "trelloupdater.js"]
            }
        ],
        "permissions": [
            "tabs",
            "storage",
            "*://*.google.com/*",
            "https://*.trello.com/*"
        ]
    }

First lets learn what these mean:

+ Meta-Data: The first 4 lines in our manifest.json provides meta-data about our extension like name, version, description.

+ Background: This is the main script that will be running for our extension. It can detect events/changes on Google Chrome and act on them. There are many events that Google Chrome provides for us to use. Read [them here](https://developer.chrome.com/extensions/runtime). This page/script will be calling other parts of our extension code to act on something. It can use chrome messaging api for this.

+ Content Scripts: Content scripts are JavaScript files that run in the context of web pages. By using the standard Document Object Model (DOM), they can read details of the web pages the browser visits, or make changes to them. We tell Chrome when to inject our content_scripts using "matches" and "permissions" keyword. Here we are telling it to inject our two content_scripts *jquery* and *trelloupdater* in all trello.com webpages. We are taking permissions to have access to chrome tabs(you will see why), chrome storage and google.com(for authentication).

> *Note that for security reasons, content scripts have different execution environment than a Web page javascript environment.*

There can be two ways for our extension to work:

+ Browser Actions: This property keeps an icon representing our extension on the right side of the address bar. Users can then click the icon and open a pop-up which is actually HTML content controlled by us. For most extensions, developers use this property.

+ Page Actions: This property is similar to browser actions but the icon is shown inside the address bar. Initially this icon is hidden and we decide when to show it. We use page actions when we need to invoke our script only when a page with specific property is loaded. Like a page with a rss feed or video.

We can load our extension using Load unpacked extension in `chrome://extensions/`. Once loaded, our background script will be loaded instantly and we can use a chrome event called [onInstalled](https://developer.chrome.com/extensions/runtime#event-onInstalled) to do something. For our example we will call [Google Oauth](https://developers.google.com/accounts/docs/OAuth2) to get the user permission for Google Drive files:

_background.js_

    function encodeData(data) {
        var urlEncodedDataPairs = [];
        for(var name in data) {
            urlEncodedDataPairs.push(encodeURIComponent(name) + '=' + encodeURIComponent(data[name]));
        }
        return urlEncodedDataPairs.join('&').replace(/%20/g, '+');
    }

    function getAccessToken(code) {
        var uriData  = {
            "client_id": "your google oauth client id",
            "client_secret": "your google client secret",
            "redirect_uri": "urn:ietf:wg:oauth:2.0:oob:auto",
            "grant_type": "authorization_code",
            "code": code
        };
        var result = false;
        var xhr = new XMLHttpRequest();
        encodedData = encodeData(uriData);
        xhr.open("POST", "https://accounts.google.com/o/oauth2/token", true);
        xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        xhr.onload = function(e) {
            if (xhr.status == 200) {
                data = JSON.parse(xhr.responseText);
                chrome.storage.sync.set({'GoogleAccess': data.access_token});
                chrome.storage.sync.set({'GoogleRefresh': data.refresh_token});
                chrome.storage.sync.set({'GoogleType': data.token_type});
                result = data.access_token;
            } else {
                console.error(xhr.statusText + ' ' + xhr.responseText);
            }
        };
        xhr.send(encodedData);
        return result;
    });

    chrome.runtime.onInstalled.addListener(function (details) {
        var uriData = {
          "access_type": "offline",
          "redirect_uri": "urn:ietf:wg:oauth:2.0:oob:auto",
          "response_type": "code",
          "client_id": "your google oauth client id",
          "approval_prompt": "auto",
          "scope": "https://spreadsheets.google.com/feeds"
        };
        var encodedData = encodeData(uriData);
        chrome.tabs.create({ url: 'https://accounts.google.com/o/oauth2/auth?'+encodedData }); //Create new tab and call Google Oauth

        chrome.tabs.onUpdated.addListener(function googleAuth(tabId, changeInfo, tab) { //Check for auth token on the page.
            if ((changeInfo.status == "complete") && (tab.title.match("Success code="))) {
            var authToken = tab.title.slice(13);
            chrome.storage.sync.set({'GoogleAuth': authToken});
            chrome.tabs.onUpdated.removeListener(googleAuth);

            //get access token
            getAccessToken(authToken);
            }
        });
    });

Here we are doing the oauth2 and getting Access token from Google. For this we have an event listener which will be called when our extension is first installed. We are doing following with our function:

+ First, we create a new tab using `chrome.tabs.create` and redirect user to Google oAuth.
+ Then, we add a listener on that tab to check when user give us permission(note that we are only taking permission for Spreadsheets) and the page have the auth token.
+ Then, we are calling function *getAccessToken* to get the access token. This is done behind the scenes.
+ At last, we are saving the access token using `chrome.storage.sync.set'.

Once we have the access token, we can make calls to Google Drive from our content scripts to get data and fill in our trello web pages using simple javascript. From our manifest, we can see that we added pattern match for our content scripts. So our scripts will be loaded on all trello web pages automatically and we can use document.onload to make the changes. 

Also we can add one more thing to our extension. When a user clicks our extension, we can explicitly call our content script again from our background page to update data. We will use chrome browser action onClicked event to send a message to our content script like this:

_background.js_

    chrome.browserAction.onClicked.addListener(function(activeTab) {
        chrome.tabs.sendMessage(activeTab.id, {action: "forceUpdate"}); // Send a message to content_scipt
    });

_trelloupdater.js_

    function getGoogleData(access_token, sheetId) {
      // Call Google Sheets API
      var sheets_url = "https://spreadsheets.google.com/feeds/list/" + sheetId + "/od6/private/full";
      $.ajax({
        type: "GET",
        url: sheets_url,
        async: true,
        headers: {
          "Authorization": "Bearer " + access_token
        }
      }).done(function (sheetXml) {
        
          //Update Trello board cards with data "sheetXml"
          $('.list-card-title').each(function() {
            var originalCard = this.text.trim().split(' ');
            var cardUuid = originalCard[originalCard.length-1];
            if ((cardUuid.length == 16) && ($.isNumeric(cardUuid))) {
              var cardName = "";
              var cardDob = "";
              $(sheetXml).find('entry').each(function () {
                if (this.textContent.search(cardUuid) != -1) {
                  cardName = $(this).find('name').text();
                  cardDob = $(this).find('dob').text();
                  return false;
                }
              });
              if (cardName) {
                this.text = cardName + " " + cardDob + " " + cardUuid;
                console.log("Changed card " + this.text);
              }
            }
          })

      }).fail(function (jqXHR, textStatus) {
        console.error(textStatus);
      });

    }

    function updateTrello(tData) {
      var page = "";
      var trelloApi = "https://api.trello.com/1/";
      var trelloBoardId = "";

      if (document.URL.indexOf(tData.board)!=-1) { 
        page = "board";
        console.log("Board Page found to sync");
        getGoogleData(tData.GoogleAccess, tData.sheet); //send sheet id and access token for Google Sheets api
      }

    }

    chrome.runtime.onMessage.addListener(function(msg) {
      if (msg.action == "forceTrello") {
        chrome.storage.sync.get(function (items) { // Get access token from chrome storage.
            updateTrello(items);
        });
      }
    });


Thats it. Our updateTrello function will be called everytime a user clicks on our extension icon. And updateTrello is just doing some simple calls to Google Sheets Api and updating trello cards according to that data. This is just tip of iceberg, a simple example showing simple event listeners like tabs.onUpdated, tabs.sendMessage, runtime.onMessage, runtime.onInstalled etc. Chrome gives you so many events, ways to [interact with your user](https://developer.chrome.com/extensions/options) that you can add complex functionalities easily. For more information, check their [documentation](https://developer.chrome.com/extensions).

