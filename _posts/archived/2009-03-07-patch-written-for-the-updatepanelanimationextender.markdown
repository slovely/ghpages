---
layout: post
title: "Patch Written for the UpdatePanelAnimationExtender"
date: 2009-03-07 0400
comments: true
disqus_identifier: 19
categories: [ASP.NET Ajax,Controls]
redirect_from: "/archive/2009-03-07-patch-written-for-the-updatepanelanimationextender.aspx/"
---
UPDATE: This patch was finally accepted!
----------------------------------------

While using the
[UpdatePanelAnimationExtender](http://www.asp.net/AJAX/AjaxControlToolkit/Samples/UpdatePanelAnimation/UpdatePanelAnimation.aspx)
control from the [Ajax Control
Toolkit](http://www.asp.net/AJAX/AjaxControlToolkit/) I decided that I
didn’t like the behaviour of the control.  My issue was that I had a
update panel that I wanted to ‘collapse’ when an async postback started,
and expand again once the postback had completed.  If you view the
controls’s
[sample](http://www.asp.net/AJAX/AjaxControlToolkit/Samples/UpdatePanelAnimation/UpdatePanelAnimation.aspx)
page you can see this effect in operation.  However, if the postback
finishes *before* the ‘collapse’ animation has finished, the animation
is aborted and the update panel will ‘jump’ to a height of zero before
expanding again.  I wanted the collapse animation to finish regardless
of how quickly the server returned to ensure that the animation always
appeared smoothly.

The way this is achieved on the sample page is by having a call to
*Thread.Sleep* in the *PageLoad* method.  I didn’t really want to waste
resources on the server just to ensure a client-side animation appeared
smoothly, so I set about writing a patch for the control.

Looking at the JavaScript behaviour for the control it was obvious why
the control behaved the way it did.  This is the JavaScript code fired
when the async postback has completed:

```csharp
    _pageLoaded : function(sender, args) {
        /// <summary>
        /// Method that will be called when a partial update (via an UpdatePanel) finishes
        /// </summary>
        /// <param name="sender" type="Object">
        /// Sender
        /// </param>
        /// <param name="args" type="Sys.WebForms.PageLoadedEventArgs">
        /// Event arguments
        /// </param>
        
        if (this._postBackPending) {
            this._postBackPending = false;
            
            var element = this.get_element();
            var panels = args.get_panelsUpdated();
            for (var i = 0; i < panels.length; i++) {
                if (panels[i].parentNode == element) {
                    this._onUpdating.quit();
                    this._onUpdated.play();
                    break;
                }
            }
        }
    }
```

As you can see, once this method is called the \_onUpdating animation is
cancelled immediately by the call to the *quit()* method.  What I needed
was a way to check that the animation has finished before playing the
\_onUpdated animation, and if not, wait until it has finished.  The
first part was easily accomplished with a simple if:

```csharp
if (this._onUpdating.get_animation().get_isPlaying()) {…}
```

The second part – waiting till it had finished – proved a bit harder
however.  My initial thought was to use window.setTimeout to check later
if the animation had finished.  However, the function supplied to
setTimeout runs in the context of the ‘window’ object, so I didn’t have
a reference to the ‘*this.\_onUpdated*’ or ‘*this.\_onUpdating*’ private
variables.  A quick Google lead me to [this page by K. Scott
Allen](http://odetocode.com/blogs/scott/archive/2007/07/04/11067.aspx)
which describes the use of the *call()* and *apply()* methods in
JavaScript.  These methods are actually on the *function* object itself
and allow us to alter what ‘this’ refers to in a method call.  Very
powerful – and definitely dangerous too – but exactly what I needed.  I
added a new private method to the JavaScript class called
*\_tryAndStopOnUpdating* as follows:

```csharp
    _tryAndStopOnUpdating: function() {
        if (this._onUpdating.get_animation().get_isPlaying()) {
            var context = this;
            window.setTimeout(function() { context._tryAndStopOnUpdating.apply(context); }, 200);
        }
        else {
            this._onUpdating.quit();
            this._onUpdated.play();
        }
    }
```

Firstly, this method checks if the first animation is still playing, and
if so uses *window.setTimeout* to wait 200ms before calling itself to
check again.  The use of ‘apply’ here ensures that when the method is
called again the ‘this’ keyword refers to our JavaScript class as
expected.  Note that if I hadn’t saved ‘this’ to a local variable and
just referred to ‘this’ in the function passed to *window.setTimeout*,
then the call would fail as ‘this’ would then refer to the JavaScript
window object itself.

All that remained was to add a new property to the server control to
allow this alternative behaviour to be switched on or off and to modify
the body of the \_*pageLoaded* method to call my new method like so:

```csharp
        if (this._postBackPending) {
            this._postBackPending = false;
            
            var element = this.get_element();
            var panels = args.get_panelsUpdated();
            for (var i = 0; i < panels.length; i++) {
                if (panels[i].parentNode == element) {
                    if (this._AlwaysFinishOnUpdatingAnimation) {
                        this._tryAndStopOnUpdating();
                    }
                    else {
                        this._onUpdating.quit();
                        this._onUpdated.play();
                    }
                    break;
                }
            }
        }
```

 

You can see an example of this modified UpdatePanelAnimationExtender
[here](http://www.simonlovely.com/demopages/updatepanelanimation/updatepanelanimation.aspx). 
The bottom checkbox controls whether the first animation will always
complete before the second one starts.  Hopefully you’ll be able to see
how much smoother the animation is with the bottom checkbox checked!

Unfortunately this patch hasn’t made it into the control toolkit yet, so
if you would like to see it in there please vote for my patch
[here](http://ajaxcontroltoolkit.codeplex.com/WorkItem/View.aspx?WorkItemId=21310). 
Thanks!

