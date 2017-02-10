---
layout: post
title: "Part II - Using TypeLite to Generate TypeScript"
date: 2014-11-07 0400
comments: true
disqus_identifier: 25
categories: [ASP.NET,.NET,TypeScript,TypeLite]
redirect_from: "/archive/2014/11/09/using-typelite-to-generate-typescript.aspx/"
---
> This is a series of posts about
> TypeLite/[TypeScript](http://www.typescriptlang.org/). The parts are:
> \
> [Part
> I](http://blog.simonlovely.com/archive/2014/11/07/typelite-has-gone-v1.0.aspx):
> TypeLite has gone v1.0 *- Video demonstrating what we are doing* \
> [Part
> II](http://blog.simonlovely.com/archive/2014/11/09/using-typelite-to-generate-typescript.aspx)
> (this part): Using TypeLite to Generate TypeScript *- Building the
> TypeScript generator* \
> [Part
> III](http://blog.simonlovely.com/archive/2015/11/16/generating-typescript-at-build-time-using-typelite.aspx):
> Generating TypeScript at build-time using TypeLite *- Automatically
> regenerating the TypeScript on each build*

With webpages becoming more interactive and feature-rich by the day,
like most developers, I’m finding more and more of my code I write is
client-side.  I’m already leveraging
[TypeScript](http://www.typescriptlang.org/) to provide type-safety
across as much of the client code as possible, but there is still a
disconnect between the TypeScript on the client, and the c\# on the
server.  If a property is renamed on the server, the compiler won’t help
me find all the places in the JavaScript that I’ve not updated (yes,
yes, of course ReSharper can help with this, but it’s not perfect).

#### There must be a better way…

What I really want is when a property is changed (renamed, deleted,
whatever) on an object that is serialised to the client, when I rebuild
I want to see any errors that it has caused in the client code.

#### One weird trick for success…

Having recently worked on a large Single Page Application, I introduced
a library called [TypeLite](https://bitbucket.org/LukasKabrt/typelite)
which enabled us to generate TypeScript definitions for all the c\#
classes that were passed over the wire.  The default use of TypeLite
uses a T4 template to generate the TS (if you want to see the normal T4
usage, [read the docs](http://type.litesolutions.net/Tutorials)).

However, this didn’t quite do what I wanted (and I just don’t like T4)
so I created a console app and using the TypeLite API directly. 

#### Here’s what I did…

*(You can follow along with my example using the repository
at*[*https://github.com/slovely/TypeScriptSample*](https://github.com/slovely/TypeScriptSample "https://github.com/slovely/TypeScriptSample")*. 
The starting point for the example is
this*[*commit*](https://github.com/slovely/TypeScriptSample/tree/7b7bac8f65613be3d28e405ea8c0da66b70268fd)*.)*

First, you’ll need to separate the objects that are sent/received by
your MVC/WebAPI actions (or, if you are crazy, your WebForms [WebMethod]
decorated static methods.  ~~You weirdo~~) into an assembly separate
from your web project.  So in my example code, I have a web project
called TypeScriptSample.Web and a class library called
TypeScriptSample.Models.  Anything that I’m passing to/from the
client/server is moved to the Models project (in my project, that’s just
one item,
[Person](https://github.com/slovely/TypeScriptSample/blob/7b7bac8f65613be3d28e405ea8c0da66b70268fd/src/TypeScriptSample.Web/Models/Person.cs)). 
*[If you are following along see
this*[*commit*](https://github.com/slovely/TypeScriptSample/commit/41ff4e8ce56bfe5bf08300db4e075587c9b7d4e1)*.]*

Next, create a new console application and use Nuget to add package
[TypeLite.Lib](https://www.nuget.org/packages/TypeLite.Lib) (it might be
easier to do this in a separate solution).  This app is going to take in
two parameters – the path to the assembly containing your models, and a
path to place the generated TypeScript.  Sample code for this is here,
but be warned this is very rudimentary and contains no error checking,
etc.  This sample takes two parameters, first one is the path of the
‘TypeScriptSample.Models’ assembly and the second is a path for the
generated TypeScript.  This should be a path in your web project.  *[See
[this
commit](https://github.com/slovely/TypeScriptSample/commit/545d6ddca094c8b03192edfb194cfd216acb83c1).]*

```csharp
using System;
using System.IO;
using System.Reflection;
using TypeLite;

namespace TypeScriptSample.Generator
{
    class Program
    {
        static void Main(string[] args)
        {
            var assemblyFile = args[0];
            var outputPath = args[1];

            LoadReferencedAssemblies(assemblyFile);
            GenerateTypeScriptContracts(assemblyFile, outputPath);
        }

        private static void LoadReferencedAssemblies(string assemblyFile)
        {
            var sourceAssemblyDirectory = Path.GetDirectoryName(assemblyFile);
            foreach (var file in Directory.GetFiles(sourceAssemblyDirectory, "*.dll"))
            {
                File.Copy(file, Path.Combine(AppDomain.CurrentDomain.BaseDirectory, new FileInfo(file).Name), true);
            }
        }

        private static void GenerateTypeScriptContracts(string assemblyFile, string outputPath)
        {
            var assembly = Assembly.LoadFrom(assemblyFile);
            // If you want a subset of classes from this assembly, filter them here
            var models = assembly.GetTypes();

            var generator = new TypeScriptFluent()
                .WithConvertor<Guid>(c => "string");

            foreach (var model in models)
            {
                generator.ModelBuilder.Add(model);
            }

            //Generate enums
            var tsEnumDefinitions = generator.Generate(TsGeneratorOutput.Enums);
            File.WriteAllText(Path.Combine(outputPath, "enums.ts"), tsEnumDefinitions);
            //Generate interface definitions for all classes
            var tsClassDefinitions = generator.Generate(TsGeneratorOutput.Properties | TsGeneratorOutput.Fields);
            File.WriteAllText(Path.Combine(outputPath, "classes.d.ts"), tsClassDefinitions);

        }
    }
}
```

To run the console app on the sample application, the command line is: \
TypeScriptSample.Generator.exe
..\\..\\..\\TypeScriptSample.Models\\bin\\debug\\TypeScriptSample.Models.dll
..\\..\\..\\TypeScriptSample.Web\\App\\server

[![image](http://blog.simonlovely.com/Images/LiveWriterUploaded/Generation-of-TypeScript-for_11FD1/image_thumb.png "image")](http://blog.simonlovely.com/Images/LiveWriterUploaded/Generation-of-TypeScript-for_11FD1/image.png)…which
produces two files in the web project (after you’ve run the command for
the first time, click show all files and include them in the web
project).  *[See [this
commit](https://github.com/slovely/TypeScriptSample/commit/4ec67d241f54e2b13caddf53a66327d2d6af1fb1)
for the results]*

 

 

 

 

Open the
[classes.d.ts](https://github.com/slovely/TypeScriptSample/blob/4ec67d241f54e2b13caddf53a66327d2d6af1fb1/src/TypeScriptSample.Web/App/server/classes.d.ts)
and you’ll find a definition of the Person object from our Models
assembly, and inside
[enums.ts](https://github.com/slovely/TypeScriptSample/blob/4ec67d241f54e2b13caddf53a66327d2d6af1fb1/src/TypeScriptSample.Web/App/server/enums.ts)
their is a translation of the server-side MaritalStatus enum!

#### Putting this to use

In the web application there’s a simple TypeScript file that retrieves a
list of Person objects from a WebAPI controller using ajax.  The current
version of this looks like:

```csharp
function getPeople() {
    $.ajax({
        url: "api/person",
        method: 'get',
        // response could be anything here
    }).done((response) => {
        var details = '<ul>';
        for (var i = 0; i < response.length; i++) {            //If 'Name' gets changed on the server, this code will fail 
            details += "<li>" + response[i].Name + "</li>";
        }
        details += '</ul>';
        $('#serverResponse').html(details);
    }).fail();
}
```

Now we can update the ‘done’ function to tell the TypeScript compiler
that the response from the server will be an array of Person objects. 
Then we get a great intellisense experience, as you can see below *[see
[this
commit](https://github.com/slovely/TypeScriptSample/commit/857dfc31e551b2020627a9226f615c4de5f18ae2)]*

[![image](http://blog.simonlovely.com/Images/LiveWriterUploaded/Generation-of-TypeScript-for_11FD1/image_thumb_3.png "image")](http://blog.simonlovely.com/Images/LiveWriterUploaded/Generation-of-TypeScript-for_11FD1/image_3.png)

 

 

 

 

 

 

 

 

That’s the basics done… However, if we add or rename a property on our
server model, we have to manually re-run the generator app to get the
TypeScript in sync.  Next time I’ll demonstrate how to integrate this as
part of your build process so that your TypeScript definitions are
updated whenever the c\# classes are modified.

