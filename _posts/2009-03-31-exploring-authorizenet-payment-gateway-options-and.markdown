---
layout: post
title:  "Exploring Authorize.net Payment Gateway Options and integrating it with django"
date:   2009-03-31 19:54:33+05:30
tags: ecommerce
author: lakshman
---
<a href="http://authorize.net/" target="_blank">Authorize.net</a> has a user base of over 200k merchants making it the largest <a class="zem_slink" title="Payment gateway" rel="wikipedia" href="http://en.wikipedia.org/wiki/Payment_gateway">payment gateway</a> service provider. Most e-commerce solutions already integrate with Authorize.net, including our favorite e-commerce store <a href="http://www.satchmoproject.com/" target="_blank">Satchmo</a>, developed in django, that <a href="http://uswaretech.com/blog/2009/03/create-your-own-online-store-in-few-hours-using-satchmo-django/">we have covered earlier.</a>

However, many shopping portals still require custom development. The robust <a href="http://en.wikipedia.org/wiki/Representational_State_Transfer">REST API</a> Authorize.net offers (AIM and SIM) allows for integration with e-commerce merchants' websites.

The AIM API allows for the check out of the customer within the merchant's site, it requires <a class="zem_slink" title="Transport Layer Security" rel="wikipedia" href="http://en.wikipedia.org/wiki/Transport_Layer_Security">SSL certificate</a> for the merchants site and data is to be transferred in an 128-bit encrypted format.

The SIM API on the other hand allows for a hosted check out from Authorize.net's site. The appearance, look and feel, CSS, logos, header and footer of their site can be customised, so that users experience a similar interface.

Thus a merchant can perform, based on his business need, one of the following:
<ol>
	<li>Using the SIM API, perform the checkout and display the receipt on Authorize.net</li>
	<li>Using the SIM API perform the checkout on Authorize.net and display the receipt on merchant's site, by using relay response</li>
	<li>Using the SIM API obtain Authorization confirmation on Authorize.net and perform the checkout and display the recipt on the merchant's site</li>
	<li>Using AIM API, perform the entire checkout process on merchant's site</li>
</ol>
The third option above is interesting. The Authorize.net's SIM provides the flexibility to checkout on merchant's site for transactions, even without SSL, for the cost of a round trip of http handshakes, just like the <a href="http://uswaretech.com/blog/2008/11/using-paypal-with-django/">Paypal's Express Checkout API</a>.

Typically, users return back to the merchant site and where the receipt is displayed (case 2) or confirm at the merchant site (case 3). Lets examine the workflow for such a scenario.
<ul>
	<li>Find the total amount payable by user, all inclusive. (incl taxes, shipping)</li>
	<li>Generate the fingerprint of transaction, based on merchant login-id, invoice-number, time-stamp and amount using the MD5 hashing library.</li>
	<li>Pre-populate all the hidden input form fields for the transaction on the template. (covered in detail, below)</li>
	<li>Send the user to Authorize.net when they submit the form, with pre-populated values</li>
	<li>After the authentication, the response is posted to merchant site</li>
	<li>Verify for success in the response.</li>
	<li>If you opted for payment on the authorize.net site itself, display the receipt.</li>
	<li>If you have opted for check out at your site, Confirm payment from the user and submit a new request of `x_type = 'PRIOR_AUTH_CAPTURE'`, also include the `x_trans_id` obtained in the response. (preferably as an AJAX request)</li>
	<li>If error occurs or if customer cancels, submit a void request.</li>
</ul>
The integration essentially involves, sending a list of hidden fields in a form to the specified url:
<ul>
	<li>Include the following Required fields in the form, set to appropriate values.</li>
</ul>

`x_fp_hash` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; The fingerprint

`x_fp_timestamp`
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
UTC time in seconds since epoch

`x_fp_sequence`
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
Invoice number, or a random number

`x_login` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Login ID of merchant, provided by Authorize.net

`x_show_form` &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
TRUE, to show form 

`x_amount` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Total Amount of the transaction
<ul>
	<li> Set the response type, `x_relay_response` to TRUE, and `url_response` to the url that to which POST has to be posted- We prefer to use the same url which is submitting the request.</li>
	<li>Set the `x_type` to `Auth_only` for a checkout at your site, or `Auth_capture` for a checkout at Authorize.net</li>
	<li>Include Additional fields, where appropriate. <a href="http://developer.authorize.net/guides/SIM/Appendix_B/Appendix_B_Alphabetized_List_of_API_Fields.htm">Entire list of fields</a></li>
	<li>Set the form submit to the Authorize.net specified url. Submissions go to: https://secure.authorize.net/gateway/transact.dll</li>
</ul>
The following view code should indicate how you should go about doing it:

<div class="highlight">
<pre><span style="font-weight: bold; color: rgb(0, 112, 32);">import</span> <span style="font-weight: bold; color: rgb(14, 132, 181);">hmac</span><span style="color: rgb(102, 102, 102);">,</span><span style="font-weight: bold; color: rgb(14, 132, 181);">time</span><span style="color: rgb(102, 102, 102);">,</span><span style="font-weight: bold; color: rgb(14, 132, 181);">urllib</span><span style="color: rgb(102, 102, 102);">,</span><span style="font-weight: bold; color: rgb(14, 132, 181);">urllib2</span>

<span style="font-weight: bold; color: rgb(0, 112, 32);">def</span> <span style="color: rgb(6, 40, 126);">payment1</span>(req):

    <span style="font-weight: bold; color: rgb(0, 112, 32);">if</span> (req<span style="color: rgb(102, 102, 102);">.</span>method <span style="color: rgb(102, 102, 102);">==</span> <span style="color: rgb(64, 112, 160);">'GET'</span>):

        <span style="font-style: italic; color: rgb(96, 160, 176);">#Use these values in the template to pre-populate a form that submits</span>
        <span style="font-style: italic; color: rgb(96, 160, 176);">#these hidded fields to url http://secure.authorize.net/gateway/transact.dll</span>
        payload <span style="color: rgb(102, 102, 102);">=</span> {
            <span style="color: rgb(64, 112, 160);">'x_login'</span> : <span style="color: rgb(64, 112, 160);">'login-id'</span>,
            <span style="color: rgb(64, 112, 160);">'x_amount'</span> : <span style="color: rgb(64, 112, 160);">'100.00'</span>,
            <span style="color: rgb(64, 112, 160);">'x_show_form'</span> : <span style="color: rgb(64, 112, 160);">'PAYMENT_FORM'</span>,
            <span style="color: rgb(64, 112, 160);">'x_type'</span> : <span style="color: rgb(64, 112, 160);">'AUTH_CAPTURE'</span>,
            <span style="color: rgb(64, 112, 160);">'x_method'</span> : <span style="color: rgb(64, 112, 160);">'CC'</span>,
            <span style="color: rgb(64, 112, 160);">'x_fp_sequence'</span> : <span style="color: rgb(64, 112, 160);">'101'</span>,
            <span style="color: rgb(64, 112, 160);">'x_version'</span> : <span style="color: rgb(64, 112, 160);">'3.1'</span>,
            <span style="color: rgb(64, 112, 160);">'x_relay_response'</span> : <span style="color: rgb(64, 112, 160);">'TRUE'</span>,
            <span style="color: rgb(64, 112, 160);">'x_fp_timestamp'</span> : <span style="color: rgb(0, 112, 32);">str</span>(<span style="color: rgb(0, 112, 32);">int</span>(time<span style="color: rgb(102, 102, 102);">.</span>time())),

            <span style="font-style: italic; color: rgb(96, 160, 176);">#The same Url as the current one, whatever it is.</span>
            <span style="color: rgb(64, 112, 160);">'x_relay_url'</span> : reverse(<span style="color: rgb(64, 112, 160);">"payment_url"</span>)
            }

        msg <span style="color: rgb(102, 102, 102);">=</span> <span style="color: rgb(64, 112, 160);">'^'</span><span style="color: rgb(102, 102, 102);">.</span>join([params[<span style="color: rgb(64, 112, 160);">'x_login'</span>],
               params[<span style="color: rgb(64, 112, 160);">'x_fp_sequence'</span>],
               params[<span style="color: rgb(64, 112, 160);">'x_fp_timestamp'</span>],
               params[<span style="color: rgb(64, 112, 160);">'x_amount'</span>]
               ])<span style="color: rgb(102, 102, 102);">+</span><span style="color: rgb(64, 112, 160);">'^'</span>

        fingerprint <span style="color: rgb(102, 102, 102);">=</span> hmac<span style="color: rgb(102, 102, 102);">.</span>new(<span style="color: rgb(64, 112, 160);">'9LyEU8t87h9Hj49Y'</span>,msg)<span style="color: rgb(102, 102, 102);">.</span>hexdigest()
        payload[<span style="color: rgb(64, 112, 160);">'x_fp_hash'</span>] <span style="color: rgb(102, 102, 102);">=</span> fingerprint

        <span style="font-weight: bold; color: rgb(0, 112, 32);">return</span> render_to_response(<span style="color: rgb(64, 112, 160);">'template1.html'</span>, payload, RequestContext(request))

    <span style="font-weight: bold; color: rgb(0, 112, 32);">else</span> <span style="font-weight: bold; color: rgb(0, 112, 32);">if</span> (req<span style="color: rgb(102, 102, 102);">.</span>method <span style="color: rgb(102, 102, 102);">==</span> <span style="color: rgb(64, 112, 160);">'POST'</span>):

        <span style="font-style: italic; color: rgb(96, 160, 176);">#Handle the response, Verify POST dictionary</span>
        <span style="font-weight: bold; color: rgb(0, 112, 32);">if</span>(req<span style="color: rgb(102, 102, 102);">.</span>post[x_response_code] <span style="color: rgb(102, 102, 102);">==</span> <span style="color: rgb(64, 160, 112);">1</span>) :
            <span style="font-style: italic; color: rgb(96, 160, 176);">#Success </span>
            <span style="font-style: italic; color: rgb(96, 160, 176);">#Display the receipt or</span>
            <span style="font-style: italic; color: rgb(96, 160, 176);">#Confirm from the user</span>
            <span style="font-weight: bold; color: rgb(0, 112, 32);">pass</span></pre>
</div>
<hr />
Looking to develop an e-commerce website? We offer services. [Get in touch](http://uswaretech.com/contact).

