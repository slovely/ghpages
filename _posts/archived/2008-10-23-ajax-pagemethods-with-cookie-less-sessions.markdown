---
layout: post
title: "Ajax PageMethods with Cookie-less Sessions"
date: 2008-10-23 0400
comments: true
disqus_identifier: 11
categories: [ASP.NET,ASP.NET Ajax]
---
Earlier this week I had to implement an auto-complete textbox (ala
Google Suggest) in a web application.  As much as I wanted to use the
newly-supported-by-MS [jQuery](http://jquery.com) framework, the
application was already using the [ASP.NET Ajax
library](http://www.asp.net/AJAX/), so I used the
[AutoCompleteExtender](http://www.asp.net/AJAX/AjaxControlToolkit/Samples/AutoComplete/AutoComplete.aspx)
control from the
[AjaxToolkit](http://www.asp.net/AJAX/AjaxControlToolkit/).

However, the implementation took longer than expected, so I thought I'd
better blog the problem so I don't spend too long debugging it the next
time it happens!

After five minutes later, I had the extender control wired into the
page, and set it to call a PageMethod - which returned a hard-coded list
of strings for testing.  However, when I ran the page I didn't get any
suggestions.  A quick check of the JavaScript console in FireFox didn't
report any errors, so I returned to the PageMethod to check that the
required attributes (*[WebMethod]*, and *[ScriptService]*) were
present.  No problems there.  Hmm, load the page again, this time with
the debugger attached to IIS and a break point in the page load. 
Nothing - the callback to the PageMethod clearly wasn't happening but no
errors on the client.  Try again in IE.  Same problem.  Argh.

Right, back to basics, so I used the fantastic
[FireBug](https://addons.mozilla.org/firefox/addon/1843) add-on to check
what was being requested from the server, and there was the problem. 
The request to the PageMethod was returning 500 - Server Error. 
However, IIS was serving this error before any of my code was being
executed.  I decided to create a simple project with the
AutoCompleteExtender so that I could test in isolation.

ARGH!  It works in the test project!  It's exactly the same!!  Then, it
twigged, the 'real' application uses cookieless sessions.  Set that in
the web.config of the test application.  Boom!  IIS returned Server
Error.  Excellent.  PageMethods don't work with SessionID's in the Url. 
Thanks. For. That.

By setting the Extender control to use a WebService instead of a
PageMethod everything worked as expected.  Now to get the wasted 2 hours
of my life back...

