---
layout: post
title: "Part I - TypeLite has gone v1.0!"
date: 2014-11-07 0400
comments: true
disqus_identifier: 26
categories: [TypeScript,.NET,TypeLite]
redirect_from: "/archive/2014/11/07/typelite-has-gone-v1.0.aspx/"
---
> This is a series of posts about
> TypeLite/[TypeScript](http://www.typescriptlang.org/). The parts are:
>
> [Part I](/archive/2014/11/07/typelite-has-gone-v1.0/) (this part): TypeLite has gone v1.0 *- Video demonstrating what we are doing*
>
> [Part II](/archive/2014/11/09/using-typelite-to-generate-typescript/): Using TypeLite to Generate TypeScript - *Building the TypeScript generator*
>
> [Part III](/archive/2015/11/16/generating-typescript-at-build-time-using-typelite/): Generating TypeScript at build-time using TypeLite - *Automatically regenerating the TypeScript on each build*

I’ve been doing a lot of work with JavaScript for the last couple of
years and have really found [TypeScript](http://www.typescriptlang.org/)
to help when working on a larger application (particularly in a team
environment).  However, it doesn’t help with the disconnect between
client-code and server-code.  If your server code is written in .NET,
I’d highly recommend checking out an awesome library from Lukas Kabrt
called [TypeLite](http://type.litesolutions.net/).  This library enables
you to generate TypeScript definitions automatically from your
server-side code!  Check out the [nuget
package](https://www.nuget.org/packages/TypeLite) now!

From the [website](http://type.litesolutions.net/):

> TypeLITE is a utility that
> generates [TypeScript](http://www.typescriptlang.org/) definitions
> from .NET classes. It supports all major features of the current
> TypeScript specification, including modules and inheritance.

I am happy to say I was able to add support for a [reasonable support of
generics](https://bitbucket.org/LukasKabrt/typelite/issue/47/handling-of-generic-classes)
and Lukas has merged my changes in and updated the version number to
v1!  To give an idea of what you can do with this I created this short
video (apologies for the production qualities – please ensure you pick
720p resolution, for some reason 1080p is grainy!!)

{% include youtubeplayer.html id="csZffHb8Sdw" %} 

My [next post](http://blog.simonlovely.com/archive/2014/11/09/using-typelite-to-generate-typescript.aspx) will
document how I achieved this, and then I’ll document how to wire it into
your build process.

