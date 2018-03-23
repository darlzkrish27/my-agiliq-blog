---
layout: post
title: "Transfer files to amazon s3 using browser instead of server"
date: 2017-04-26 02:29:55+05:30
tags:  aws
draft: true
---
So basically when we try to upload a file to amazon s3, we will do it in a regular way of 

1. Getting the file from frontend
2. Passing it to server
3. Make a connection to amazon s3 using python code
4. Upload file to amazon s3

But you should also know that, we have a way of uploading directly from browser to s3 instead of server

We are going to show that how to do it, and we are using django, boto, html, jquery, ajax to achieve this

Hope all the above requirements are already installed and your django site is up and running

### Concept of uploading to s3

1. In order to upload a file to amazon s3 we need to generate a signed url using amazon's python boto package.
2. Next step is to make the uploaded file public in order for to access it.

**Lets start designing it**

### Create a template called **upload_s3.html** in your tempalte directory

	<!DOCTYPE html>
	<html>
	<head>
	    <meta charset="UTF-8">
	    <meta name="description" content="amazon">
	</head>
	<body>
		<form action="#" method="post" enctype="multipart/form-data">
		    Select image to upload:
		    <input type="file" name="fileToUpload" id="fileToUpload">
		    <input class="amazon_upload" type="submit" value="Upload File" name="submit">
		</form>
	<script type="text/javascript" src="{ static 'js/amazon/amazon.js' }"></script>
	<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.0/jquery.min.js"></script>
	</body>
	</html>

### Create django urls to generate the signed url and make them public

**project/urls.py**

	from django.conf.urls import url, include
	from .views import GetS3SignedUrl, Makes3VideoPublic

	urlpatterns = [
		url(r'^generate_signed_url/$', GetS3SignedUrl.as_view(), name='generate_signed_url'),
	    url(r'^make_video_public/$', Makes3VideoPublic.as_view(), name='make_video_public'),
	]

### Create django views to generate the signed url and make them public

You need to copy your **AWS_ACCESS_KEY_ID**, **AWS_SECRET_ACCESS_KEY**, **S3_BUCKET_NAME** in to your settings.py file

**project/views.py**
	
	from django.views import View
	from django.conf import settings
	from django.http import HttpResponse

	import os
	import boto
	from boto.s3.connection import S3Connection

	def get_s3_connection(use_host=False):
	    key = getattr(settings, 'AWS_ACCESS_KEY_ID', None)
	    secret = getattr(settings, 'AWS_SECRET_ACCESS_KEY', None)
	    if not key or not secret:
	        return None
	    if use_host:
	        return S3Connection(key, secret, host='s3.amazonaws.com')
	    return S3Connection(key, secret)

	class GetS3SignedUrl(View):
	    """
	    Generate Signed url for s3
	    """

	    def get(self, request, *args, **kwargs):
	        os.environ['S3_USE_SIGV4'] = 'True'
	        c = get_s3_connection(use_host=True)
	        file_name = request.GET.get('file_name')
	        username = request.GET.get('username')
	        final_file_name = 'videos/{0}/{1}'.format(username, file_name)
	        secondsPerDay = 24*60*60
	        url = c.generate_url_sigv4(
	            secondsPerDay, "PUT", bucket=settings.S3_BUCKET_NAME, key=final_file_name, force_http=True)
	        out_url = 'https://%s.s3.amazonaws.com/%s' % (settings.S3_BUCKET_NAME, final_file_name)
	        del os.environ['S3_USE_SIGV4']
	        return HttpResponse({'signed_request': url, 'url': out_url, 's3_key': final_file_name, 'status': 'ok'})

	class Makes3VideoPublic(View):
	    """
	    Make s3 video public
	    """
	    def post(self, request, *args, **kwargs):
	        post_data = request.POST
	        aws_access_key_id = settings.AWS_ACCESS_KEY_ID
	        aws_secret_access_key = settings.AWS_SECRET_ACCESS_KEY
	        conn = boto.connect_s3(aws_access_key_id, aws_secret_access_key)
	        try:
	            bucket = conn.get_bucket(settings.S3_BUCKET_NAME, validate=True)
	        except S3ResponseError:
	            bucket = conn.create_bucket(settings.S3_BUCKET_NAME)
	        k = bucket.get_key(request.POST.get('s3_key'))
	        k.make_public()
	        print post_data
	        return Response(post_data)

### Let's create js file 'js/amazon/amazon.js' in your js directory to make calls to the endpoints


	$( document ).ready(function() {
	   $('.amazon_upload').click(function(){
	   	// Get the file from frontend
	   	var myFile = $('#fileToUpload').prop('files');
	   	// Send a get request to generate signed url
	   	var config = {
	         params: {
	            file_name : myFile.name,
	            username : 'Username'
	         },
	         headers : {'Accept' : 'application/json'}
	        };

	    $http.get('generate_signed_url/', config).then(
	        function(response) {
	            self.signed_data = response.data
	            var url = response.data.signed_request

	            // Upload file directly to amazon s3
	            var xhr = new XMLHttpRequest();
	            xhr.open("PUT", url);
	            xhr.setRequestHeader('Access-Control-Allow-Headers', '*');
	            xhr.upload.onprogress = updateProgress;
	            xhr.onerror = function() {
	                alert("Could not upload file.");
	            };
	            xhr.send(self.video);

	            function updateProgress (ev) {
	                    if (ev.lengthComputable) {
	                        var percentComplete = Math.round((ev.loaded / ev.total) * 100);
	                        console.log(percentComplete);
	                    }
	                }

	            // Make posted url / video as public
	            var s3_config = {
	                params : response.data,
	                headers : {'Accept' : 'application/json'}
	            }
	            var data = {
	                signed_request : response.data.signed_request,
	                s3_key : response.data.s3_key,
	                status : response.data.status,
	                url : response.data.url,
	            }
	            $.ajax({
	              type: "POST",
	              url: 'make_video_public/',
	              data: data,
	              success: function(response){
	                console.log(response);
	              },
	            });

	         },function(response) {}
	         );

	   })

	})


So from the above js code we are making two ajax calls, one is for generating the signed request with below response data

	{
		signed_request: "https://xxxxxxxx.s3.amazonaws.com/videos/xxxxxx…EKQLHA%2F20170425%2Fus-east-1%2Fs3%2Faws4_request", 
		url: "https://xxxxxxx.s3.amazonaws.com/videos/shivaagiliq/FunnyCat.mp4", 
		s3_key: "videos/shivaagiliq/FunnyCat.mp4", 
		status: "ok"
	}

and another is to make the video public

After this when you navigate to amazon file/video url you can able to access it

Finally we achieved posting the file from browser to amazon s3 directly


