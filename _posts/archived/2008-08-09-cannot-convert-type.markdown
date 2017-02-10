---
layout: post
title: "Cannot convert type &lsquo;ASP.login_aspx&rsquo; to &lsquo;System.Web. UI.WebControls.Login&rsquo;"
date: 2008-08-09 0400
comments: true
disqus_identifier: 4
redirect_from: "/archive/2008/08/09/cannot-convert-type.aspx/"
categories: [ASP.NET]
---
Ran into this problem with a website last week.  It slowed me down for a
while so I’m posting about it here to help others (or more likely me,
when I run into it again!).

The problem is caused by having an aspx page which has the same name as
a System.Web.UI control.  The auto-generated class will try to reference
the System.Web.UI control when it should be referencing your page
class.  So, it happens for Login.aspx, ChangePassword.aspx etc.

The solution I found on the net is to simply to rename your page. 
However, this isn’t ideal as it will obviously break any existing links
to the page.  A better fix is to modify the code-behind class name
instead.  e.g. Change this:

\<%@ Page Language=”C\#” AutoEventWireup=”true” CodeFile=”Login.aspx.cs”
Inherits=”Login”  %\>

To this:

\<%@ Page Language=”C\#” AutoEventWireup=”true” CodeFile=”Login.aspx.cs”
Inherits=”Login***Page***“  %\>

Then you need to rename the class in your code behind file to
LoginPage.  This means that the page will work again, and any existing
links to Login.aspx will also still work!

