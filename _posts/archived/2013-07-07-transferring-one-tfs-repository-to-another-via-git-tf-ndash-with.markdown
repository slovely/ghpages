---
layout: post
title: "Transferring one TFS repository to another via GIT-TF &ndash; with history!"
date: 2013-07-07 0400
comments: true
disqus_identifier: 23
categories: [Git,TFS]
redirect_from: "/archive/2013-07-07-transferring-one-tfs-repository-to-another-via-git-tf-ndash-with.aspx/"
---
Recently a client needed to migrate a large TFS repository to a new
machine, and to a later version of TFS.  They tried to follow the
Microsoft procedure but had problem with that (different OS versions,
security settings, that sort of thing).  In the end they decided to just
‘Get Latest’ from the old repo and commit that into the new one, losing
all the history of the source code.

As retrieving history / comparing old versions of code, is one of the
main jobs of a source code provider, I suggested using GIT-TF to do the
migration.  After a fair bit of googling I had a stab at doing the
import.  As it took me a few attempts and none of the instructions were
quite right (at least in our scenario) I thought I’d post a demo of the
complete instructions here. (Prerequisites – you must have a working GIT
prompt and have successfully installed
[GIT-TF](http://gittf.codeplex.com/).  These instructions assume that
you are using GIT Bash).

##### Current TFS Repositories

Our two TFS histories look like this (Old on the left, new on the
right).  Of course, in reality the history on the left would be much
bigger.  Notice that the latest commit on the new repository is removing
all the files that TFS automatically adds – the build process templates
etc.  You should also do this, as we want to start with the new
repository empty. \
[![image](http://blog.simonlovely.com/Transferring-one-TFS-repository-to-anoth_1153D/image_thumb.png "image")](http://blog.simonlovely.com/Transferring-one-TFS-repository-to-anoth_1153D/image.png)

[![image](http://blog.simonlovely.com/Transferring-one-TFS-repository-to-anoth_1153D/image_thumb_3.png "image")](http://blog.simonlovely.com/Transferring-one-TFS-repository-to-anoth_1153D/image_3.png)
\
Our new TFS server looks the same, but has no history apart from the
auto-generated check-in's of the TF Build Automation and template
files.  **You should delete this files from the New TFS repository now
(and remember to check-in the deletes!).**

##### 

##### Clone the TFS repository's to GIT

Run these commands in a GIT prompt:

```csharp
cd c
mkdir git
cd git
git tf clone http://myoldserver:8080/tfs $/OldTfs --deep
git tf clone http://mynewserver:8080/tfs $/NewTfs --deep
```

This will create two new GIT repositories under c:\\git called OldTfs
and NewTfs.  The NewTfs git repository should be empty, as per your new
TFS repository.  Running git log on the OldTfs git repo should display
your complete TFS checkin history.

##### Remove link between GIT repository and TFS Changesets

[![git-tf
file](http://blog.simonlovely.com/Transferring-one-TFS-repository-to-anoth_1153D/git-tf-file_thumb.jpg "git-tf file")](http://blog.simonlovely.com/Transferring-one-TFS-repository-to-anoth_1153D/git-tf-file.jpg)

Now, as we need to pull in the ‘old’ GIT repository to the new one, we
need to remove the details of the new TFS changesets that we’ve already
pulled into the GIT repo.  To do that, remove the file “git-tf” from the
“.git” folder in c:\\git\\NewTfs.

Now we need to re-create the link to the new server (but without the
changeset details), so run this command:

```csharp
cd c/git/NewTfs 
git tf configure http://mynewserver:8080/tfs $/NewTfs
```

##### Pull in the old GIT repository and push to new TFS

Next we need to add the old GIT repo as a remote in the new one, and
then pull from it.  The important option is to specify “--rebase” to
ensure that the full commit history is pulled across:

```csharp
git remote add master file:///c/git/OldTfs
git pull --rebase master master
```

Running “git log” should now display the full history of your old TFS
repository in the new GIT repo, so the only step left is to push this to
your new TFS server:

```csharp
git tf checkin --deep
```

Remember the “--deep” option or only the latest changeset will be
committed.  Once this is finished, you should be able to see your full
TFS history displayed in the Source Control Explorer on your new server!
\


