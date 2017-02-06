---
layout: post
title: "Using Castle's Dynamic Proxy Part 2 - Using Mixins"
date: 2008-11-02 0400
comments: true
disqus_identifier: 13
categories: [Patterns]
redirect_from: "/archive/2008-11-02-using-castles-dynamic-proxy-part-2-using-mixins.aspx/"
---
As promised, this is my follow up post to [this
post](http://blog.simonlovely.com/archive/2008/09/28/using-castles-dynamic-proxy-to-enable-lazy-loading.aspx "Using Castle's Dynamic Proxy to Enable Lazy Loading"). 
This time I will show how to use the
[DynamicProxy](http://www.castleproject.org/dynamicproxy/index.html)
library with [mixins](http://en.wikipedia.org/wiki/Mixin).  Using
mixin's you can add functionality to an object at runtime.  In this
example, I will continue the lazy load theme from the previous post.

Imagine we have a contacts application, which contains a Person object. 
This all works splendidly until one day we are asked to create a sales
application.  This application must use our existing repository of
contacts, but cannot use the same database.  In this new application we
derive a new class from Person, with a few new attributes defining how
the customer is to be treated:

```csharp
public class Customer : Person
{
    public bool ApplyDiscount { get; set; }
    public bool AllowFreeDelivery { get; set; }
}
```

Now, to load a Customer, we must first load the main person data from
the contacts database, and then load the extended data from the new
sales database.  This means that each time a Customer object is loaded,
there will be two database calls (no, we can't use linked servers to
join the databases!).  Most of the time, this would be fine, but if you
don't need the extended Customer information, then it's still a wasted
database call.  *(I know this sounds like a very contrived example, but
I have really worked on a project where we needed to do just this).*

How can we get round this?  Well, in a similar way to the previous post,
we need to dynamically load the data when it's requested.  In the
previous example however, when we intercept to 'get' call, we check to
see if the data has already been loaded to prevent it hitting the
database every time we get the property.  However, in this case the
Customer object only has boolean fields - we can't check these to see if
the data has already been loaded as neither true nor false imply that we
haven't loaded the properties from the database.  We could change the
properties to use nullable boolean's (*bool?*  in c\#), then the lazy
load method could check that one of the properties was null before
loading the data.  Alternatively, we could add a boolean field called
*isLoaded* to the class and check that instead.

The problem with both of these solution though is that we would then
have data-access concerns in our entity model.  This is not good
[SOC](http://en.wikipedia.org/wiki/Separation_of_concerns)!  The
solution we are going to use is to add the isLoaded flag *at runtime!* 
We will not have to make any modifications to the Customer class defined
above.  To do this, we need to add the following interface and
implementation:

```csharp
public interface ILazyLoad
{
    bool IsLoaded { get; }
}

public class LazyLoadImpl : ILazyLoad
{
    bool _isLoaded;
    public bool IsLoaded
    {
        get { return _isLoaded; }
    }
}
```

Now we need our Customer object to implement this interface, so when we
create a new object we instead tell Castle to create us a proxy (as per
previous post), but also add the ILazyLoad interface:

```csharp
//Create a proxy object instead of a standard Customer object.
Castle.DynamicProxy.ProxyGenerator a = new Castle.DynamicProxy.ProxyGenerator();
//Create an instance of our Lazy Load Implementation and pass that to Castle.
ILazyLoad mixin = new LazyLoadImpl();
ProxyGenerationOptions pgo = new ProxyGenerationOptions();
pgo.AddMixinInstance(mixin);
Customer c = a.CreateClassProxy(typeof(Customer), pgo, new CustomerInterceptor());
```

Now our CustomerInterceptor can intercept calls to the 'get' properties
of the Customer object, then cast the Customer instance to ILazyLoad and
check the value of the IsLoaded property to see if the data needs to be
loaded.  If it does, then the data is retrieved and the properties are
set.  The \_isLoaded field is then set to 'true' using reflection.  Next
time we access a property, the interceptor will know that we've already
loaded the data so won't need to hit the database again.  We now have a
Customer object that loads it's data only when required.  (Please refer
to the [previous
post](http://blog.simonlovely.com/archive/2008/09/28/using-castles-dynamic-proxy-to-enable-lazy-loading.aspx)
to see how to setup the interceptors).

 

**Note:***  This example uses DynamicProxy v1.  I believe version 2 of
DynamicProxy works in the same way, but having not personally used it I
can't guarantee this.*

