---
layout: post
title:  "Importing your old comments to Disqus site"
date:   2015-01-16 19:42:08+05:30
tags: Disqus
author: Rakesh
---
In one of my latest [blogpost](http://agiliq.com/blog/2014/11/disqus-and-disqus-sso/) on disqus I covered topics on integrating Disqus to the website and disqus SSO. In this post, I will let you know how to migrate the older comments to Disqus. 

If you sneak peek in to the alluring features of disqus you may make your mind to migrate your custom commenting system on your blog to use disqus commenting system. The threaded comments and replies, powerful moderation and admin tools, RSS options and many more features come in as battaries included with Disqus which makes the commenting more interactive and easy to deal with.

Let us kick start the process. Till date importing the old comments directly from the blogger and wordpress to disqus is feasible. This can be achieved by using tools and plugins that are already existing in them and its pretty straight forward. In this post our prime concern will be laid on custom XML import format. If you are using neither blogger, nor wordpress the custom XML import format which is based on the WXR (WordPress eXtended RSS) schema comes to the rescue. Disqus also supports MovableType, and IntenseDebate but preferably WXR is of more concern.

I consider a case in a Django project, which does adopt a custom commenting system in which the comments are stored in a postgresql database. This process of migrating the comments to disqus comprises of two steps in this case.

1. Creating a WXR schema of the existing comments
2. Importing the WXR file to Disqus site.

1. Creating a WXR schema of the existing comments
-----------------------------
WXR format is based on the Rss specification. This format is normally used for importing or exporting data from wordpress servers. Now the aim is to create a WXR file from the existing comments in our storage. For that we are going to write a python script. Disqus has prescribed a format of the file that we are planning to import to Disqus. The format looks in the following way.

    <?xml version="1.0" encoding="UTF-8"?>
    <rss version="2.0"
      xmlns:content="http://purl.org/rss/1.0/modules/content/"
      xmlns:dsq="http://www.disqus.com/"
      xmlns:dc="http://purl.org/dc/elements/1.1/"
      xmlns:wp="http://wordpress.org/export/1.0/"
    >
      <channel>
        <item>
          <!-- title of article -->
          <title>Foo bar</title>
          <!-- absolute URI to article -->
          <link>http://foo.com/example</link>
          <!-- body of the page or post; use cdata; html allowed (though will be formatted to DISQUS specs) -->
          <content:encoded><![CDATA[Hello world]]></content:encoded>
          <!-- value used within disqus_identifier; usually internal identifier of article -->
          <dsq:thread_identifier>disqus_identifier</dsq:thread_identifier>
          <!-- creation date of thread (article), in GMT. Must be YYYY-MM-DD HH:MM:SS 24-hour format. -->
          <wp:post_date_gmt>2010-09-20 09:13:44</wp:post_date_gmt>
          <!-- open/closed values are acceptable -->
          <wp:comment_status>open</wp:comment_status>
          <wp:comment>
            <!-- sso only; see docs -->
            <dsq:remote>
              <!-- unique internal identifier; username, user id, etc. -->
              <dsq:id>user id</dsq:id>
              <!-- avatar -->
              <dsq:avatar>http://url.to/avatar.png</dsq:avatar>
            </dsq:remote>
            <!-- internal id of comment -->
            <wp:comment_id>65</wp:comment_id>
            <!-- author display name -->
            <wp:comment_author>Foo Bar</wp:comment_author>
            <!-- author email address -->
            <wp:comment_author_email>foo@bar.com</wp:comment_author_email>
            <!-- author url, optional -->
            <wp:comment_author_url>http://www.foo.bar/</wp:comment_author_url>
            <!-- author ip address -->
            <wp:comment_author_IP>93.48.67.119</wp:comment_author_IP>
            <!-- comment datetime, in GMT. Must be YYYY-MM-DD HH:MM:SS 24-hour format. -->
            <wp:comment_date_gmt>2010-09-20 13:19:10</wp:comment_date_gmt>
            <!-- comment body; use cdata; html allowed (though will be formatted to DISQUS specs) -->
            <wp:comment_content><![CDATA[Hello world]]></wp:comment_content>
            <!-- is this comment approved? 0/1 -->
            <wp:comment_approved>1</wp:comment_approved>
            <!-- parent id (match up with wp:comment_id) -->
            <wp:comment_parent>0</wp:comment_parent>
          </wp:comment>
        </item>
      </channel>
    </rss>

* Things to keep in mind:

As recommended by the Disqus we need some points to keep in mind while creating a WXR file of the existing comments.

1. Only use the "dsq:remote" tag (and its children) if you're using Single Sign-On; otherwise remove it.
2. If you're manually filling in <wp:comment_author_email> make sure it's unique to each user or else all comments will display the same author name.
3. Each "item" tag can parent an infinite number of "wp:comment" tags.
4. There is a minimum character requirement for each comment message, which is 3 characters.

Every item is supposed to be a comment on the site. As I am not using the SSO in the site, we can remove the <dsq:remote>. The link specifies the site url to the specific post. In my case I happened to find some encoding issues with the <content> which does take the whole body of the post so I preferred removing it from the WXR file and it worked fine. 

The script appears to be in the below manner. I accessed my comment model to get the comment data. And then I created the xml file in the format prescribed by Disqus. I skipped some tags which ought to mess up encryption issues. But beware, try to create as the disqus prescribes for the easy flow. The script looks like the following way.

    from mysite.models import Comment
    from django.contrib.sites.models import Site
     
    site = Site.objects.all()[0]
     
    comments = Comment.objects.all()
    xml_data_header = """<?xml version="1.0" encoding="UTF-8"?>
                    <rss version="2.0"
                      xmlns:content="http://purl.org/rss/1.0/modules/content/"
                      xmlns:dsq="http://www.disqus.com/"
                      xmlns:dc="http://purl.org/dc/elements/1.1/"
                      xmlns:wp="http://wordpress.org/export/1.0/"
                    >
                          <channel>
                      """
    xml_data_footer = """
                          </channel>
                        </rss>
                      """
    file_dis = open('disqus.xml','wb')
    file_dis.write(xml_data_header)
    for comment in comments:
        xml_data_all = ''
        xml_data = """
                    <item>
                      <title>{0}</title>
                      <link>http://{1}</link>
                      <content:encoded><![CDATA[{2}]]></content:encoded>
                      <dsq:thread_identifier>{3}</dsq:thread_identifier>
                      <wp:post_date_gmt>{4}</wp:post_date_gmt>
                      <wp:comment_status>open</wp:comment_status>
                      <wp:comment>
                        <wp:comment_id>{5}</wp:comment_id>
                        <wp:comment_author>{6}</wp:comment_author>
                        <wp:comment_author_email>{7}</wp:comment_author_email>
                        <wp:comment_author_url>{8}</wp:comment_author_url>
                        <wp:comment_author_IP>{9}</wp:comment_author_IP>
                        <wp:comment_date_gmt>{10}</wp:comment_date_gmt>
                        <wp:comment_content><![CDATA[{11}]]></wp:comment_content>
                        <wp:comment_approved>1</wp:comment_approved>
                      </wp:comment>
                    </item>

                    """.format(comment.comment_for.title.encode('utf-8'), site.domain+comment.comment_for.get_absolute_url().encode('utf-8'), comment.comment_for.text.raw.encode('utf-8'), comment.comment_for.pk, comment.comment_for.created_on, comment.pk, \
                        comment.user_name.encode('utf-8'), comment.email_id.encode('utf-8'), comment.user_ip, comment.created_on, comment.text.encode('utf-8'), \
                        )
        # xml_data_all += xml_data
        file_dis.write(xml_data)
     
    file_dis.write(xml_data_footer)
    file_dis.close()

Once you run the above script the resultant will be a "disqus.xml" file. An example output of the file looks like this:

    <?xml version="1.0" encoding="UTF-8"?>
                <rss version="2.0"
                  xmlns:content="http://purl.org/rss/1.0/modules/content/"
                  xmlns:dsq="http://www.disqus.com/"
                  xmlns:dc="http://purl.org/dc/elements/1.1/"
                  xmlns:wp="http://wordpress.org/export/1.0/"
                >
                      <channel>                  
                        <item>
                          <title>Google Appengine - First Impressions</title>
                          <link>http://agiliq.com/blog/2008/04/google-appengine-first-impressions/</link>
                          <content:encoded><![CDATA[245]]></content:encoded>
                          <dsq:thread_identifier>245</dsq:thread_identifier>
                          <wp:post_date_gmt>2008-04-09 04:02:57</wp:post_date_gmt>
                          <wp:comment_status>open</wp:comment_status>
                          <wp:comment>
                            <wp:comment_id>482</wp:comment_id>
                            <wp:comment_author>Ivan</wp:comment_author>
                            <wp:comment_author_email>suchy.ivan@gmail.com</wp:comment_author_email>
                            <wp:comment_author_url></wp:comment_author_url>
                            <wp:comment_author_IP>None</wp:comment_author_IP>
                            <wp:comment_date_gmt>2008-04-09 08:48:53</wp:comment_date_gmt>
                            <wp:comment_content><![CDATA[<p>On dev server data is stored in /tmp folder</p>]]></wp:comment_content>
                            <wp:comment_approved>1</wp:comment_approved>
                          </wp:comment>
                        </item>

                         </channel>
                </rss>
Once we are done with creating the WXR schema we should import it to the disqus site.

2.Importing the WXR file to Disqus site.
------------------------------------

To attain the second part you need to have a disqus account and a disqus site available for you. You can get to know more about creating a disqus site in the linked [blogpost](http://agiliq.com/blog/2014/11/disqus-and-disqus-sso/) 

Once you are done with the creation of disqus site go to the settings which can be accessible on admin panel. There appears the settings page, click on the discussion on the right top panel. Click "import" tab and there appears the different types of import formats that disqus supports. Select generic(WXR) and click on browse. Upload the disqus.xml file and once the process get completed it shows up if there are any errors in the uploaded file. If there are any errors, the process of importing comments will be kept on hold. If the comments are successfully imported it takes some time to get those comments reflected in your site.

TIP: The best practice would be to try importing in to test disqus site and once it works perfectly, move towards importing them to the main site.




