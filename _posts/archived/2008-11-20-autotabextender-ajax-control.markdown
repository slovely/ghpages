---
layout: post
title: "AutoTabExtender AJAX Control"
date: 2008-11-20 0400
comments: true
disqus_identifier: 14
categories: [ASP.NET Ajax,Controls]
redirect_from: "/archive/2008/11/20/autotabextender-ajax-control.aspx/"
---
I have just uploaded my AutoTabExtender control to my [Projects
page](http://www.simonlovely.com/Projects.aspx).  Please feel free to
take a look at the
[code](http://www.simonlovely.com/ProjectDetail.aspx?id=1) and download
it if it's useful for you.

I decided to create this control when I had a broken hand.  While
entering my memorable to login to my online bank account, I realised
that having to press 'tab' to move from day to month to year was really
slowing me down with only one typing hand!!  This control can extend a
textbox so that focus automatically moves on when the length of the text
entered equals the
[MaxLength](http://msdn.microsoft.com/system.web.ui.webcontrols.textbox.maxlength)
property.  The usage of the extender is as follows:

```csharp
<asp:TextBox runat="server" ID="txtPart1"MaxLength="2" Columns="2"></asp:TextBox>
<asp:TextBox runat="server" ID="txtPart2"MaxLength="2" Columns="2"></asp:TextBox>
<asp:TextBox runat="server" ID="txtPart3"MaxLength="2" Columns="2"></asp:TextBox>

<spl:AutoTabExtender runat="server" ID="ate1" TargetControlID="txtPart1" NextControlID="txtPart2"></spl:AutoTabExtender>

<spl:AutoTabExtender runat="server" ID="ate2" TargetControlID="txtPart2" NextControlID="txtPart3"></spl:AutoTabExtender>
```

 

So, you need an extender for each textbox that should auto-tab.  You
then need to specify the NextControlID to indicate to which control the
focus should be moved.  In the example above I have used a form for
entering a sortcode, so we need an extender for the first and second
textboxes.  The extender will also allow 'natural' deleting between
textboxes.  See the live demo for this
[here](http://www.simonlovely.com/DemoPages/AutoTabExtenderDemo.aspx).

The extender should work in IE, FF and Chrome.

