---
layout: post
title:  "Google diff match patch library"
date:   2014-05-09 16:29:34+05:30
tags: google
author: manjunath
---

Diff match patch library is useful to compare the differences between the two texts.It compares the texts and displays what is added, removed or unchanged.

We use `Python` in our example below.


#### Let us see a quick example

Firstly, we will install diff match patch

    pip install diff-match-patch

Moving forward we will import diff_match_patch

    import diff_match_patch

Let us consider using two strings, we will name it as `old_string` and `new_string`

    import diff_match_patch    
    
    old_string = """I'm selfish, impatient and a little insecure. I make mistakes,
    I am out of control and at times hard to handle. But if you can't handle me at my worst,
    then you sure as hell don't deserve me at my best."""

    new_string = """I'm selfish, impatient and a little secure. I don't make mistakes,
    I am out of control and at times hard to handle difficult things. But if you can't handle me at my worst,
    then you sure as hell don't deserve me at my best."""


Now, we will create an object of `diff_match_patch` and call `diff_main(old_string, new_string)`

    import diff_match_patch    
    
    old_string = """I'm selfish, impatient and a little insecure. I make mistakes,
    I am out of control and at times hard to handle. But if you can't handle me at my worst,
    then you sure as hell don't deserve me at my best."""

    new_string = """I'm selfish, impatient and a little secure. I don't make mistakes,
    I am out of control and at times hard to handle difficult things. But if you can't handle me at my worst,
    then you sure as hell don't deserve me at my best."""

    diff_obj = diff_match_patch.diff_match_patch()
    diffs = diff_obj.diff_main(old_string, new_string)

#### What `diff_main(text1, text2)` does?

It computes an array of differences to describe the transformation of text1 into text2.
For Example in the above code `diffs` contains an array of tuples.

We will see what this tuple contains, before that let us make this array Human readable.

`diff_match_patch` has a method `diff_cleanupSemantic(diffs)` which does the job for us.

    diff_obj.diff_cleanupSemantic(diffs)

Now variable `diffs` contains

`[(0, "I'm selfish, impatient and a little "),
 (-1, 'in'),
 (0, 'secure. I '),
 (1, "don't "),
 (0, 'make mistakes,\nI am out of control and at times hard to handle'),
 (1, ' difficult things'),
 (0, ". But if you can't handle me at my worst,\nthen you sure as hell don't deserve me at my best.")]`


Now it looks better, let us see what these tuples contains:

* The first element specifies if it is an insertion (1), a deletion (-1) or an equality (0).
* The second element specifies the affected text. 

To convert these array of tuples to HTML.We use `diff_prettyHtml(diffs)`.

    html = diff_obj.diff_prettyHtml(diffs) 

There you are, we can now see the differences in old and new text.

** <span>I\'m selfish, impatient and a little </span><del style="background:#ffe6e6;">in</del><span>secure. I </span><ins style="background:#e6ffe6;">don\'t </ins>   
<span>make mistakes,&para;<br>I am out of control and at times hard to handle</span><ins style="background:#e6ffe6;"> difficult things</ins><span>. But if you can\'t
handle me at my worst,&para;<br>then you sure as hell don\'t deserve me at my best.</span> **

But the above html is not so clear to understand what is added and removed right?

We will write a method to see these changes side by side.

    import diff_match_patch

    old_string = """I'm selfish, impatient and a little insecure. I make mistakes,
    I am out of control and at times hard to handle. But if you can't handle me at my worst,
    then you sure as hell don't deserve me at my best."""

    new_string = """I'm selfish, impatient and a little secure. I don't make mistakes,
    I am out of control and at times hard to handle difficult things. But if you can't handle me at my worst,
    then you sure as hell don't deserve me at my best."""

    class SideBySideDiff(diff_match_patch.diff_match_patch):

        def old_content(self, diffs):
            """
            Returns HTML representation of 'deletions'
            """
            html = []
            for (flag, data) in diffs:
                text = (data.replace("&", "&amp;")
                        .replace("<", "&lt;")
                        .replace(">", "&gt;")
                        .replace("\n", "<br>"))
    
                if flag == self.DIFF_DELETE:
                    html.append("""<del style=\"background:#ffe6e6;
                        \">%s</del>""" % text)
                elif flag == self.DIFF_EQUAL:
                    html.append("<span>%s</span>" % text)
            return "".join(html)
    
        def new_content(self, diffs):
            """
            Returns HTML representation of 'insertions'
            """
            html = []
            for (flag, data) in diffs:
                text = (data.replace("&", "&amp;")
                        .replace("<", "&lt;")
                        .replace(">", "&gt;")
                        .replace("\n", "<br>"))
                if flag == self.DIFF_INSERT:
                    html.append("""<ins style=\"background:#e6ffe6;
                        \">%s</ins>""" % text)
                elif flag == self.DIFF_EQUAL:
                    html.append("<span>%s</span>" % text)
            return "".join(html)


    diff_obj = SideBySideDiff()
    result = diff_obj.diff_main(old_string, new_string)
    diff_obj.diff_cleanupSemantic(result)

    old_record = diff_obj.old_content(result) 
    new_record = diff_obj.new_content(result) 


Here is the output for the above code:

<div style="margin-left: 5%; float: left; width: 45%; overflow: auto; display: block; background-color: #fff;<br />}">
<p>
<h2>Old record</h2>

<span>I'm selfish, impatient and a little </span><del style="background:#ffe6e6;">in</del><span>secure. I </span><span>make mistakes,<br>I am out of control and at times hard to handle</span><span>. But if you can't handle me at my worst,<br>then you sure as hell don't deserve me at my best.</span>

</p>

</div>



<div style="margin-left: 5%; float: left; width: 45%; overflow: auto; display: block; background-color: #fff;<br />}">
<p>
<h2>New record</h2>
<span>I'm selfish, impatient and a little </span><span>secure. I </span><ins style="background:#e6ffe6;">don't </ins><span>make mistakes,<br>I am out of control and at times hard to handle</span><ins style="background:#e6ffe6;"> difficult things</ins><span>. But if you can't handle me at my worst,<br>then you sure as hell don't deserve me at my best.</span>

</p>
</div>

For reference and some more methods see [API](http://code.google.com/p/google-diff-match-patch/wiki/API)

Feedbacks and suggestions are welcome :)


e :)


