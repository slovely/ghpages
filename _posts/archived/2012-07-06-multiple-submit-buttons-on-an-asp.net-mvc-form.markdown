---
layout: post
title: "Multiple submit buttons on an ASP.NET MVC form"
date: 2012-07-06 0400
comments: true
disqus_identifier: 22
categories: [MVC,Patterns]
redirect_from: "/archive/2012/07/06/multiple-submit-buttons-on-an-asp.net-mvc-form.aspx/"
---
Wow, well over 3 years since a blog post!

I recently needed to create a form containing multiple buttons. 
Normally, I use a variation of [this
technique](http://blog.maartenballiauw.be/post/2009/11/26/Supporting-multiple-submit-buttons-on-an-ASPNET-MVC-view.aspx)
to know which button is clicked, and have each handled by a different
action method.  However, on this occasion the button was actually the
same button repeated for a list of entities, so mapping by name wasn’t
good enough – each button was named “edit”.  I needed a way to know
*which* edit button was pressed.  In this instance having a \<form\> for
each button plus a hidden input specifying which entity was being edited
wasn’t acceptable – each entity also had other input controls that
needed to be submitted as one, and had to work without JavaScript.

So I created
[MultiButtonExAttribute](https://github.com/slovely/MultiButtonEx/blob/master/src/MultiButtonEx.Web/MultiButtonExAttribute.cs)
(an MVC ActionNameSelector) which matched only on the prefix of the
button name, and used the rest of the name to store state information. 
All you have to do is create input buttons using this pattern:

```csharp
<input type="submit" name="edit_id:1234_other:somestring" value="Edit" />
```

Where the name is made up of a prefix (“edit”), then a separator (“\_”),
then key/value pairs of data separated by a colon.  Each key/value pair
is then separated by another underscore.  On the server-side, create an
action method to handle the form submit and decorate it like this:

```csharp
[MultiButtonEx("edit")]
[HttpPost]
public ActionResult EditEntity(int id, string other)
{
    //TODO: whatever needs to be done
    //The ID will be parsed for you by the DefaultModelBinder
    //and in this case will have the integer value 1234
}
```

Note that the key/value pairs take part in the normal model binding, so
are passed type-safe to the parameters of the action method.

To make the submit button easier to render, I also created a
[HtmlHelper](http://https://github.com/slovely/MultiButtonEx/blob/master/src/MultiButtonEx.Web/SubmitButtonExtension.cs)
which ensures the ‘name’ attribute is generated correctly:

```csharp
@Html.MultiButtonEx(new {id = item.Id, other = item.Other}, "edit", "Click Me!")
```

Which will translate the anonymous object into the correct format.

NOTE: The code on github is an example and not production ready – you’ll
no doubt want to beef up the error handling, move the separator
characters into consts and encode those characters if they appear in
your data, etc.  Also, I’m sure there’s no doubt a limit on the length
of a HTML name attribute (which probably just for fun varies across
browsers).

I am also not even sure this is a good idea – if anyone can think of a
better way to achieve this please let me know!!

More info on [github
repository](https://github.com/slovely/MultiButtonEx).

