---
layout: post
title: "Adjust SQL Connection String at runtime"
date: 2014-05-27 0400
comments: true
disqus_identifier: 24
categories: [.NET,Hacks]
redirect_from: "/archive/2014/05/27/adjust-sql-connection-string-at-runtime.aspx/"
---
Recently I needed the ability to modify the current database for an
application at runtime.  There are lots of ways of doing this, for
example the IDbConnection interface defines the
[ChangeDatabase](http://msdn.microsoft.com/en-us/library/system.data.idbconnection.changedatabase(v=vs.110).aspx)
method allowing you to do just that.  Alternatively, you will have
abstracted away your connection object behind a factory or inject it in
using your [favourite IoC tool](http://docs.structuremap.net/).

However, I was faced with some old code that created the SqlConnection
object as needed in *hundreds* of different places, and didn’t have the
opportunity to go through and replace all of these references, so looked
at modifying the ConfigurationManager.ConnectionStrings collection
directly.  I thought that would be easy enough, but the base
ConfigurationElement class has a read-only flag preventing
modification.  There’s always a way though… as long as you use
reflection you can indeed modify the connection string!

```csharp
//Update the readonly flag to false, using reflection:
var settings = ConfigurationManager.ConnectionStrings["MyConnectionName"];
var fieldInfo = typeof(ConfigurationElement).GetField("_bReadOnly", BindingFlags.Instance | BindingFlags.NonPublic);
fieldInfo.SetValue(settings, false);

//Create a connection string builder as it makes it easy to modify just the DB name:
var builder = new SqlConnectionStringBuilder(ConfigurationManager.ConnectionStrings["MyConnectionName"].ConnectionString);
builder.InitialCatalog = dbName;  //You can also change, server, user, password, etc here, if required
//Update the connection string setting:
settings.ConnectionString = builder.ConnectionString;
```

Any new connections created after this will use the new connection
string!  In my case, only the database name needed to be changed, so I
only set the InitialCatalog, but you can set anything else you need as
well.

Note that this is NOT a sensible way to do things – accessing private
data can break in future releases or cause unintended side-effects.  In
my case however, this code was only used for debug builds (and wrapped
in \#if DEBUG…) so it was good enough, YMMV.

