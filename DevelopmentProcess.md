# Target users #

This document is for programmers who want to contribute changes to ELMAH.

# Version control #

## Overview ##

ELMAH uses Mercurial, a distributed version control system. What this means is that, even though this page hosts a central repository, there can be many clone repositories with changes of their own, and then some of those can be merged back into the main repository.

The model for developing for ELMAH is the following:
  1. Each developer creates an google code hosting [clone of the main ELMAH repository](http://code.google.com/p/elmah/source/checkout). This [clone is hosted on Google servers](http://code.google.com/p/elmah/source/clones).
  1. The developer then makes a local clone of his code hosting clone, which is then at his local machine.
  1. The developer writes new code into his local clone and commits it locally
  1. When a change is ready to be integrated back into the main repository, that change is pushed from the developer's local clone to his code hosting clone
  1. The developer then requests a code review by opening a [new issue](http://code.google.com/p/elmah/issues/entry), saying which clone has the code to be reviewed, what it's supposed to do, and what are the relevant changesets
  1. The code will be reviewed on the user's clone - if any further changes are suggested, the process repeats from (3)
  1. Once the change is approved, a member of the ELMAH team will merge it back into the main repository

Even though this may sound complicated, this process makes code reviews easy and allows a lot of people to work on changes in parallel.

Next is an overview of each step, but if you want to really learn Mercurial, please look at the references at the bottom of this document.

## Mercurial installation ##

First, make sure you have Mercurial installed by running the command:

```
$ hg version
Mercurial Distributed SCM (version 2.0.1)
(see http://mercurial.selenic.com for more information)

Copyright (C) 2005-2011 Matt Mackall and others
This is free software; see the source for copying conditions. There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

If you don't want your hostname and username to be made public, you can change how you're identified in commits you make by editing your ~/.hgrc file:

```
[ui]
username = John Doe
```

## Making a clone of the repository ##

We'll need to create two clones of the main ELMAH repository - one online, and then a local clone of that one.

To create the online clone, click on **Source** above, then on **Create Clone**. Give your clone a name, summary and description, then click on **Create repository clone**. At that point the online clone is ready.

**IMPORTANT**
If you plan to have your code reviewed, then you also want to go into **Administer**, **Source**, and check **Allow non-members to review code**.

To create the local clone, click on **Source** tab of your clone page, and then use the checkout command provided there:

```
hg clone https://johndoe-elmah.googlecode.com/hg/ johndoe-elmah
```

Optionally, you can add your username and password to it (so you don't have to type them in every time):

```
hg clone https://johndoe:mypassword@johndoe-elmah.googlecode.com/hg/ johndoe-elmah
```

and that's it - you have a local copy of your clone (in this example, in subdirectory `johndoe-elmah`) which you can then make changes to.

## Bringing in new changes from the master repository ##

The recommended way of bringing changes in from the main repository is the use of `hg fetch`:

```
$ hg fetch http://elmah.googlecode.com/hg/
```

Please note that the fetch command is a Mercurial extension which is equivalent to `hg pull -u` plus `hg merge` plus `hg commit` - for more details please see the references, but this basically means that it will try to merge the incoming changes with your local changes.

If you want to see what will be brought in with the above command before running it, you can use:

```
$ hg incoming
```


## Committing changes locally ##

Commiting changes locally is easy - run `hg status` to see the state of your local clone:

```
$ hg status
?  MyNewFile
M  MyChangedFile
!  MyDeletedFile
```

In the above example, it shows one file that it knows nothing about (MyNewFile), one that it knows about but is missing (MyDeletedFile) and one that has had changes made to it.

To add all the previously unknown files and remove any missing files, use the addremove command:
```
$ hg addremove
Adding MyNewFile
Deleting MyDeletedFile
```

hg status then shows the new status:

```
$ hg status
A  MyNewFile
M  MyChangedFile
D  MyDeletedFile
```

If you wish to see what has changed, you can use the `hg diff` command.
Finally, you can commit the changes with

```
$ hg commit
```

which will open an editor for you to type in a description for these changes. Optionally, you can specify filenames to hg commit in order to commit only part of your current changes.

**IMPORTANT**: When your change is pulled into the main ELMAH source, the change description that you entered here will show up as [changes in the main ELMAH source](http://code.google.com/p/elmah/source/list), so please use a meaninful description - "fixing bug", "making changes", etc. are not ok, please instead use something like "fixing GPX import bug caused by null pointer", "adding Russian translation", etc. so that it makes sense in the context of ELMAH as a whole, not just your clone.

## Pushing changes to your online clone ##

Pushing changes to your online clone is incredibly simple:

```
$ hg push
pushing to https://johndoe:***@johndoe-elmah.googlecode.com/hg/
searching for changes
adding changesets
adding manifests
adding file changes
added 1 changesets with 1 changes to 1 files
```

and you're done.

If you want to see what changes you're going to push before you do it, you can also use the following command:

```
$ hg outgoing
comparing with https://johndoe:***@johndoe-elmah.googlecode.com/hg/
searching for changes
changeset:   5:b6fed4f21233
tag:         tip
user:        John Doe
date:        Tue May 05 06:55:53 2009 +0000
summary:     Added an extra line of output
```

## Requesting a code review ##

To request a code review, [open a new Issue in ELMAH](http://code.google.com/p/elmah/issues/entry), select template **Review request**, fill out the fields from the template. Someone will then review the code changes and integrate them when ready.

# References #

[Mercurial: The definitive guide](http://hgbook.red-bean.com/)


---


This wiki was adapted from the [Development Process wiki of mytracks project](http://code.google.com/p/mytracks/wiki/DevelopmentProcess).