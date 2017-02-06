---
layout: post
title: "Post-Redirect-Get Pattern in MVC"
date: 2008-11-26 0400
comments: true
disqus_identifier: 16
categories: [ASP.NET,MVC]
redirect_from: "/archive/2008-11-26-post-redirect-get-pattern-in-mvc.aspx/"
---
I found a good write-up of the
[PRG](http://blog.eworldui.net/post/2008/05/ASPNET-MVC---Using-Post2c-Redirect2c-Get-Pattern.aspx)
pattern in MVC by [Matt
Hawley](http://blog.eworldui.net/author/matthaw.aspx) this week, and
have decided to use it in an MVC project I'm working on.  I have made a
few changes to Matt's code however. 

##### 1 - Use ModelBinders and ModelState to Simplify Code

Firstly, as the new version of MVC (the beta release) supports [Model
Binders](http://weblogs.asp.net/scottgu/archive/2008/10/16/asp-net-mvc-beta-released.aspx),
I updated the example to use these instead.  Now we can just save the
ModelState into TempData in one go, instead of saving the error messages
and the users input, so the Submit action looks something like:

```csharp
public ActionResult Submit()
{
    //OMITTED: Do work here...
    if (!ModelState.IsValid)
    {
        //Save the current ModelState to TempData:
        TempData["ModelState"] = ModelState;
    }
}
```

In the Create action we just need to pull these values out of TempData
and add to the ModelState.  MVC will then enter the users input back
into the textboxes for you.  The Create action looks like:

```csharp
public ActionResult Create()
{
    //If we have previous model state in TempData, add it to our
    //current ModelState property.
    var previousModelState = TempData["ModelState"] as ModelStateDictionary;
    if (previousModelState != null)
    {
        foreach (KeyValuePair<string, ModelState> kvp in previousModelState)
            if (!ModelState.ContainsKey(kvp.Key))
                ModelState.Add(kvp.Key, kvp.Value);
    }

    return View();
}
```

(You will want to wrap this boiler plate code up in a helper class or
something though)

##### 2. Fix Scenario where user input will be lost

Secondly, I did find a small issue with the code as posted, when the
user does the following:

> GET the page with a form \
> POST the form with invalid input \
> REDIRECT back to the page (with the users input intact) \
> REFRESH the page - the users input is now lost!

I'm not sure this is a particularly common scenario, but losing the
users input is never a good way to instil trust in your application! 
The reason the data is lost in this case is because we stored the users
input in TempData which only exists for this request and the next one. 
I thought about putting the value into Session instead, but then you'd
have to come up with a strategy for removing the items at the right time
(you wouldn't want the form to remember it's values from the last time
it was used for instance).  In the end I decided that just putting the
values *back into*****TempData would be the best solution.  This
requires the following line to be added to the Create action:

```csharp
public ActionResult Create() 
{ 
    //If we have previous model state in TempData, add it to our 
    //current ModelState property. 
    var previousModelState = TempData["ModelState"] as ModelStateDictionary; 
    if (previousModelState != null) 
    { 
        foreach (KeyValuePair<string, ModelState> kvp in previousModelState) 
            if (!ModelState.ContainsKey(kvp.Key)) 
                ModelState.Add(kvp.Key, kvp.Value); 
        TempData["ModelState"] = ModelState;  //Add back into TempData
    } 

    return View(); 
}
```

If the user now refreshes the page after a validation failure, they will
no longer lose their input.  If they go on to fix the validation errors
and submit the form, the saved TempData value will be automatically
cleared by the MVC framework.

