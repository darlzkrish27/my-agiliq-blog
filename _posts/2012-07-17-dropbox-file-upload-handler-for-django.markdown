---
layout: post
title: "Dropbox file upload handler for django"
date:   2012-07-17 00:00:00
author:   thejaswi
tags:   upload
---
Dropbox
[announced](http://blog.dropbox.com/index.php/new-dropbox-pro-plans/)
new pro plans last week and some accounts have had their storage size
doubled. Wouldn\'t it be wonderful if we could upload all our files to
dropbox from our django webapp?

In this post, I write a custom file upload handler that will upload
files from our application to dropbox.

Let us see how to use the custom file upload handler.

Install the [Dropbox Python
SDK](https://www.dropbox.com/developers/reference/sdk) before you setup
your django app to handle the file uploads.

In your settings.py, add the following attributes (with the values
filled):

    DROPBOX_APP_KEY = ""
    DROPBOX_APP_SECRET_KEY = ""
    DROPBOX_APP_ACCESS_TOKEN = ""
    DROPBOX_APP_ACCESS_TOKEN_SECRET = ""

    # Optional values below

    # The folder where you want the files uploaded.
    # Example: /Public or /
    DROPBOX_FILE_UPLOAD_FOLDER = ""
    # The value below may be either 'app_folder' or 'dropbox'
    DROPBOX_ACCESS_TYPE = ""

The DROPBOX\_APP\_KEY and DROPBOX\_APP\_SECRET\_KEY are provided to you
when you [create a new dropbox
app](https://www.dropbox.com/developers/apps). Fetching the access token
and access token secret is outside the scope of this blog post but you
can follow the [Getting Started
Guide](https://www.dropbox.com/developers/start/authentication#python)
until the Get an access token section and then paste the access token
key and secret in the DROPBOX\_APP\_ACCESS\_TOKEN and
DROPBOX\_APP\_ACCESS\_TOKEN\_SECRET attributes respectively.

Add the [DropboxFileUploadHandler](https://gist.github.com/3128835) to
any app (in my case testapp) and reference it in the
FILE\_UPLOAD\_HANDLERS in \`settings.py\`:

    FILE_UPLOAD_HANDLERS = (
        "testapp.dropbox_upload_handler.DropboxFileUploadHandler",
    )

That\'s it and you are done!

::: {.note}
::: {.admonition-title}
Note
:::

Since dropbox doesn\'t support chunked uploads, the file is first
uploaded to the temporary file upload directory on the server and then
onto dropbox.
:::

Here\'s how you would handle the file post upload in your view:

    def file_upload_handler_view(request):
        if request.method == "POST":
            file_uploaded = request.FILES["name_of_file_input"]
            print file_uploaded.read()
        # Helpful attribute to get dropbox file metadata
        # like path on the server, size, thumbnail etc
        file_uploaded.dropbox_metadata

The DropboxFile returned is an instance of
[httplib.HTTPResponse](http://docs.python.org/library/httplib.html?highlight=httplib#httplib.HTTPResponse)
and so all file like methods are not defined but some basic methods like
read are supported. There is an attribute called dropbox\_metadata on
the uploaded file that holds the [dropbox
metadata](https://www.dropbox.com/developers/reference/api#metadata-details).
