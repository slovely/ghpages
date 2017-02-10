---
layout: post
title: "Using Castle's Dynamic Proxy to Enable Lazy Loading"
date: 2008-09-28 0400
comments: true
disqus_identifier: 8
categories: [Patterns]
redirect_from: "/archive/2008/09/28/using-castles-dynamic-proxy-to-enable-lazy-loading.aspx/"
---
Many enterprise-level applications try to use POCO classes for their
entities to enable a clean separation of concerns between business logic
and data access.  One of the challenges of this is when an object
contains a collection of child objects which should be loaded only when
required.

A traditional approach to this would be to check whether the collection
needed to be loaded in the 'getter' for the property.

```csharp
public class Order
{

    IList<OrderItems> _items;
    
    public IList<OrderItem> Items
    {
        get
        {
            if (_items == null)
            {
                //Load the order items collection here...
                //Something like OrderItemsRepository.Load(this.ID)...
            }
            return _items;
        }
    }

}
```

The issue with this approach is that the entity now has a direct
dependency on the data access layer.  Suddenly our class isn't so POCO! 
The class is now much harder to test in isolation, as the data access
methods will need to be mocked.

An alternative approach to this is to use an AOP technique to inject the
data access methods at runtime.  The idea here, is that the
*OrderRepository* returns a proxy *Order* object that knows how to load
the OrderItems collection.  In this example I will use
[DynamicProxy](http://www.castleproject.org/dynamicproxy/index.html)
from the [Castle Project](http://www.castleproject.org/).

##### Step 1 - Remove the data access code from the Order object

Change your entity to remove any code that try's to load the child
objects.  Your final class should look something like:

```csharp
public class Order
{

    IList<OrderItems> _items;
    
    public IList<OrderItem> Items
    {
        get
        {
            return _items;
        }
    }

}
```

Don't worry that the \_*items* field is never instantiated - that will
be rectified later.

##### Step 2 - Modify the *OrderRepository* to use a proxy object

Somewhere in your repository a new *Order* object will be created and
it's properties set from the data store.  This will need to be changed
to create a new proxied *Order* object instead.

This:

```csharp
public Order LoadOrder(int id)
{
    Order o = new Order();
    //Load the order from the data store
    o.ID = ....;
    o.OrderDate = ....;
    //etc...
    return o;
}
```

Becomes this:

```csharp
public Order LoadOrder(int id)
{
    //Create a proxy object instead of a standard Order object.
    Castle.DynamicProxy.ProxyGenerator a = new Castle.DynamicProxy.ProxyGenerator();
    Order o = a.CreateClassProxy(typeof(Order), new OrderInterceptor(), null);

    //Load the order from the data store
    o.ID = ....;
    o.OrderDate = ....;
    //etc...
    return o;
}
```

***Important Note***: In a real application, the *ProxyGenerator* object
should only be created once per application instance - or you will
rapidly run out of resources!  (Yes, I did learn that one from
experience!).

The second parameter to *CreateClassProxy* specifies the class that will
end up loading the *OrderItems* for us.  This will be covered in the
next step.

##### Step 3 - Create the interceptor class

The interceptor class contains a method, *Intercept()*,  that will be
called every time a method on the base class (in our case the *Order*
class) is called.  Into this method will be passed all the details of
the method that was called on the base class, and any parameters that
were passed to it.  Using this information, we can hook into the getter
of the *Order.Items* property and load the *OrderItem* collection at
this point.

The implementation of our interceptor will be something like this:

```csharp
public class OrderInterceptor : Castle.DynamicProxy.StandardInterceptor
{

    public override object Intercept(Castle.DynamicProxy.IInvocation invocation, params object[] args)
    {
        //Get the object on which the method was called
        Order o = invocation.InvocationTarget as Order;
        //Check the method name is one in which we are interested
        if (invocation.Method.Name == "get_Items")
        {            
            LoadOrderItems(o);
        }

        //At the end of the method, call the base implementation which will ensure the original 
        //method is invoked, by which point the _items field will have been loaded.
        base.Intercept(invocation, args);
    }

    private void LoadOrderItems(Order o)
    {
        //Get the _items field from the Order object
        IList<OrderItem> items = typeof(Order).GetField("_items", BindingFlags.NonPublic | BindingFlags.Instance).GetValue(o);
        //Check if we have already loaded the items. If not, load them
        if (items == null)
        {
            items = Factory.OrderItemRepository.LoadOrderItemsByOrderId(o.ID);
            //Set the private field _items in the Order object
            IList<OrderItem> items = typeof(Order).GetField("_items", BindingFlags.NonPublic | BindingFlags.Instance).SetValue(o, items);
        }
    }

}
```

You can now try running this and you should find that when your client
code calls *Order.Items*, the collection is retrieved from the database
and returned in the *IList*.  **However, if you try this now you will
get a null reference exception!  The next step explains what to do and
why!**

##### Step 4 - Make the methods we need to intercept *virtual*

This is a very easy step, the reason for me including it separately is
that it is also an easy one to forget to do!

The reason for the null reference exception is that the interceptor we
lovingly crafted earlier wasn't even called!  Therefore the *\_items*
field will always remain null.* ***Why wasn't it called?**  Well, the
generated proxy *Order* object inherits from the real *Order* object and
overrides all of it's method to allow the interceptor to be called. 
However, our *Items* collection could not be overridden as it was not
virtual.  Adding the virtual keyword will make this example work as
expected.

In a future post I will discuss using mixin's for more advanced
scenarios.

