---
layout: post
title: "Alternative Stylesheets for Different Browsers"
date: 2009-02-22 0400
comments: true
disqus_identifier: 18
categories: []
redirect_from: "/archive/2009/02/22/alternative-stylesheets-for-different-browsers.aspx/"
---
On a site I've been working on recently various [Css
Hacks](http://www.webcredible.co.uk/user-friendly-resources/css/hacks-browser-detection.shtml)
are used to ensure that the site is displayed consistently on all
browsers.  However, as more and more browsers are released the css files
just got messier and harder to maintain.  Moving the 'hacks' into their
own file and selectively including them would make life a lot easier. 
The usual way to achieve this is using [conditional
comments](http://www.quirksmode.org/css/condcom.html).  This is only
supported in IE, but as most of our css hacks were around IE this was
acceptable.

The problem with this however, is that the site was using ASP.Net
Themes, and that automatically adds the relevant stylesheets to the page
for you - meaning that you have no way of selectively choosing the
correct stylesheets!  *(Incidentally, I'd love to be proved wrong about
this so please let me know if I'm missing something!).*

I decided to write a more flexible theming system instead.  The plan was
to load all the stylesheets in a certain directory and add them to the
pages automatically in the same way ASP.Net themes do.  But it would
also support convention-based subdirectories containing the 'hacks' for
the different browsers.  The structure would be something like this:

[![image](/images/LiveWriterUploaded/AlternativeStylesheetsforDifferentBrowse_11DBF/image_thumb.png)](/images/LiveWriterUploaded/AlternativeStylesheetsforDifferentBrowse_11DBF/image.png)

Any css files in the **Theme1** directory would always be included, but
css files in the **IE** directory would only be included if the user was
using IE.  The convention for the names of the folder is to match the
Browser property of the
[HttpBrowserCapabilities](http://msdn.microsoft.com/en-us/library/system.web.httpbrowsercapabilities_members.aspx)
class (accessible from Request.Browser).  I ended up also allowing
further sub-directories so that different browser *version*s could have
different stylesheets.


[![image](/images/LiveWriterUploaded/AlternativeStylesheetsforDifferentBrowse_11DBF/image_thumb_3.png)](/images/LiveWriterUploaded/AlternativeStylesheetsforDifferentBrowse_11DBF/image_3.png)

If you need a stylesheet for a specific version of a browser, you just
create a folder with the version number as its name.  e.g. To have a
stylesheet specifically for FireFox v2, create a folder called '2' in
the FireFox folder.  If you want a stylesheet for IE versions 6 and
below, you can place it in a folder called '6-'.  Likewise, if you want
a stylesheet for versions 7 and up, you should place it in a folder
called '7+'.  In the future I may extend this convention to allow things
like '1-3' and '4-7' so that ranges of versions can be included.

I have uploaded this theming engine
[here](http://simonlovely.com/ProjectDetail.aspx?id=6).  To use the
engine you must register the StylesheetManager control on your
webforms/masterpage like so:

> \<%@ Register Assembly="SPL.WebSite.Projects"
> Namespace="SPL.WebSite.Projects.WebControls.Theming" TagPrefix="spl"
> %\>

And then in the \<head /\> section include an instance of the control:

> \<spl:StylesheetManager runat="server"
> ThemeDirectory="\~/DemoPages/DemoStyles" /\>

The only property you need to set is the location of the root directory
of your theme.  When the control renders it will figure out which
stylesheets are required based on the user's browser and write out
\<link /\> tags for each one.

When running in release mode, instead of linking to *n* stylesheets, the
control will link to an HttpHandler instead which will merge the css
files into one and write them directly into the response.  To get this
working you need to include this handler in your web.config:

> \<add verb="GET" path="CssCombiner.axd"
> type="SPL.WebSite.Projects.HttpHandlers.Theming.CssCombineHandler,
> SPL.WebSite.Projects"/\>

Note that the handler will cache the css to avoid multiple disk accesses
on each request.  Currently this is cached for a hard-coded time of 1
hour.  Depending on your circumstances you may wish to change this to
use a configuration value instead.

Feel free to use this theming engine if it meets your needs and please
let me know if you have any improvements.  Note that the uploaded
version doesn't contain things like error handling, logging, etc and the
http handler it uses hard-coded.  These are all things you will probably
want to modify before using in anger.

A demo page is available
[here.](http://simonlovely.com/demopages/themingenginedemo.aspx)

