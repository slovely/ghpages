---
layout: post
title: "Part III - Generating TypeScript at build-time using TypeLite"
date: 2015-11-16 0400
comments: true
disqus_identifier: 27
categories: [.NET,TypeScript,TypeLite,ASP.NET]
redirect_from: "/archive/2015/11/16/generating-typescript-at-build-time-using-typelite.aspx/"
---
> This is a series of posts about
> TypeLite/[TypeScript](http://www.typescriptlang.org/). The parts are:
> 
> [Part I](/archive/2014/11/07/typelite-has-gone-v1.0/): TypeLite has gone v1.0 *- Video demonstrating what we are doing*
>
> [Part II](/archive/2014/11/09/using-typelite-to-generate-typescript/): Using TypeLite to Generate TypeScript *- Building the TypeScript generator*
>
> Part III (this part): Generating TypeScript at build-time using TypeLite *- Automatically regenerating the TypeScript on each build*

Well, it’s been over a year since I started a series on using
[TypeLite](http://type.litesolutions.net/) for improving the type-safety
of your client side code.  At least no one reads this, so it doesn’t
matter!  This is now part III.  On the plus side, it does mean that over
a year later I’m still using this technique in a number of applications
that have gone live, and it’s proved it’s worth.

In the [previous episode](http://blog.simonlovely.com/archive/2014/11/09/using-typelite-to-generate-typescript.aspx),
 we had a solution setup in which we could regenerate our TS interfaces
by running a command.  Now to make it generate on each build.

### Step One

First step, we need to copy the TypeScriptGenerator EXE to a sensible
location.  Unfortunately running it from the location it’s built in can
cause file locking issues with VisualStudio.  As this project doesn’t
change very often, I add a simple batch file to the project like this 
(see [this commit](https://github.com/slovely/TypeScriptSample/commit/7a708dc8541899365bee0e1cdf37ff2fb004be1f)):

    copy bin\debug\*.dll ..\Tools
    copy bin\debug\*.exe ..\Tools
    del ..\Tools\*.vshost.exe
    pause

\*Note, I use the VSCommand extensions
([http://vscommands.squaredinfinity.com/](http://vscommands.squaredinfinity.com/ "http://vscommands.squaredinfinity.com/"))
which add a ‘Run’ option on the solution explorer context menu for batch
files, making this workflow ok with me.  *Currently I’m committed the
cardinal sin of then checking this ‘Tools’ folder into git with the
required binary exe/dlls.  Feel free to move this step to your build
script so that you can avoid this.*

### Step Two

Now we need to call the generator each time the solution is compiled. 
Initially I used Build Events to do this, but I learned that these fire
at a different time if you run instead Visual Studio compared to running
outside using MSBuild, so it had trouble when building on a Build
Server.  After quite a bit of trial and error, I settled on using an
MSBuild extensibility point – add this to the csproj file of your Web
project (see [this commit](https://github.com/slovely/TypeScriptSample/commit/000fdd37e1968e2ed12ad0b7965997cfb6cd15e5)):

```csharp
<Project>
  <!-- the rest of the project file -->
  <Target Name="AfterResolveReferences">
    <Exec Command="..\Tools\TypeScriptSample.Generator.exe ..\TypeScriptSample.Models\bin\$(ConfigurationName)\TypeScriptSample.Models.dll          $(ProjectDir)App\server" WorkingDirectory="$(ProjectDir)" />
  </Target>
</Project> 
```

[AfterResolveReferences](https://msdn.microsoft.com/en-GB/library/ms366724.aspx)
runs at the perfect time for us.  The references for the Web project
have been pulled in, so the DLL containing the c\# model classes will be
in the \\bin folder, but it’s *before* the web project itself is built –
so any client-side errors introduced by the updated TypeScript (e.g. a
property is renamed) will be reported as usual and prevent the build
from succeeding.  The command call is just the same command that we were
running manually in [part
two](/archive/2014/11/09/using-typelite-to-generate-typescript/).

### The final result

Now, if we rename a property in the c\# code: \
[![image](/images/LiveWriterUploaded/70167857f96f_EC01/image_thumb.png "image")](/images/LiveWriterUploaded/70167857f96f_EC01/image.png)


All we have to do is build, and any TypeScript that references the old
property name will appear as an error!

[![image](/images/LiveWriterUploaded/70167857f96f_EC01/image_thumb_3.png "image")](/images/LiveWriterUploaded/70167857f96f_EC01/image_3.png)


In the year since I started this series, the version of the generator
I’m using actually does a lot more than just convert c\# models into TS
interfaces – it generates typed SignalR hubs, and also creates type-safe
TypeScript method calls for WebAPI actions, allowing you to call your
server-side methods from the client with intellisense for action
parameters, etc.  I’m hoping I’ll be able to tidy that up and add it to
this series of posts.

