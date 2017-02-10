---
layout: post
title: "Be careful when using ScriptManager. RegisterClientScriptResource"
date: 2008-10-30 0400
comments: true
disqus_identifier: 12
categories: [ASP.NET]
redirect_from: "/archive/2008/10/30/be-careful-when-using-scriptmanager.-registerclientscriptresource.aspx/"
---
I ran into a problem with registering a client script resource this week
which had me confused for a while.  I had an existing custom control
which registered a script resource and worked as expected.  When I
created a new custom control (in another assembly) derived from this
control however, I kept getting a ''Resource xyz.js could not be
found''.  The line in the base class that registered the script looked
like:

```csharp
ScriptManager.RegisterClientScriptReource(this.Page, this.GetType(), "xyz.js");
```

The reason this didn't work when using the derived control was because
the 2nd parameter is used by .NET to find the assembly containing the
resource, so it (incorrectly) looked in the assembly containing the
derived class instead!  The fix is simply to change the line above to
explicitly specify the correct type instead:

```csharp
ScriptManager.RegisterClientScriptResource(this.Page, typeof(MyBaseCustomControl), "xyz.js");
```

The resource will then be picked up from the correct assembly.

