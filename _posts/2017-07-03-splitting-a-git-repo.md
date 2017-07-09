---
title: "Splitting a Git Repository"
permalink: "/2017/06/splitting-a-git-repository.html"
date: "2017-06-03 20:15:01"
updated: "2017-06-03 20:15:01"
excerpt: "A quick reference for splitting a git repository into two while preserving the history."
header:
  teaser: "/assets/images/git-icon.png"
categories: [git, repository, tips, maintenance ]
comments: true
---

## Intro

There are times when you need to split your code base into smaller more maintainable pieces. For example when you decide to make a shared library out of some of your code. Or that handful of images has turned into an entire catalog and is bloating the repository.

You could just move the folder containing the library files into a new git repository, but you would loose the entire history of development that has occurred, making it harder to find and fix bugs, and keep track of changes. This post will show you how to split a git repository and preserve the history of the files in both repositories.

## Notes before we begin

* This process applies to one branch, in this example it will be the master branch. I'm assuming you're using one of the most common git [work flow development models](http://nvie.com/posts/a-successful-git-branching-model/) or some derivative of it where the master branch contains the latest stable release.

* Once this process is complete *all* the hashes will be re-written, and anyone with a copy of the repository will have to delete there copy and clone the new repositories to continue working on the project. If they have their own branches that they are working on they'll need to create patches and apply them against the new repositories.

* Because you have to delete the existing repository and rewrite the hashes you'll need to find a time when you can do the transition that does not interfere with your teams development work, or schedule the interruption ahead of time.

* Most of the process goes quickly but step 9 can take awhile on large repositories, so I'd recommend doing at least one dry run to figure out how much time you'll need to complete the process for your project. On a repository just over 1GB in size step 5 took nearly 2 days.

## The Example Repo

We have a repository containing our software. Inside of it we have a libs/ and docs/ directory that we want to split into another repository, and we want to put the apps/ directory into a repository by itself.

* /software/libs/
* /software/app/
* /software/docs/

## Break it up

1) First clone your master repository:
    {% highlight bash %}
    git clone ssh://git@myserver.com/software.git
    cd software/
    {% endhighlight %}

2) Create a new branch that we can work on. Using `git filter-branch` split out the first directory we're going to move to the new repository.

    {% highlight bash %}
    git checkout -b shared-libs
    git filter-branch --prune-empty --subdirectory-filter /libs
    {% endhighlight %}

This will put the content of /libs folder at the top level of the repository. So if you want to reorganize the folder do it now and commit your changes to the branch.

If there are other folders you want to move into the new repository you need to checkout the master branch again and repeat the steps above using a different branch. So for our example repository above:

    {% highlight bash %}
    git checkout master
    git checkout -b shared-docs
    git filter-branch --prune-empty --subdirectory-filter /docs -f
    {% endhighlight %}

The -f flag is needed for each subsequent run of `git filter-branch` because git saves the objects that you have removed.

Again the contents of the /docs directory will be at the top level so reorganize the files as desired and commit your changes to the branch.

3) Now that we have our code separated out we need to create a new repository to keep it in. Create a directory, and an empty repository just like normal.

    {% highlight bash %}
    cd ../
    mkdir new-libs-repo
    cd new-libs-repo
    git init
    {% endhighlight %}

4) Import all the branches you created above.

    {% highlight bash %}
    git pull ../software shared-libs
    {% endhighlight %}

For each additional branch you created in step 2 above, use this line to import it into this repository. The command below will merge the history of the additional branch(es) together.

    {% highlight bash %}
    git pull ../software another-folder --allow-unrelated-histories
    {% endhighlight %}

If you didn't make changes to the file structure in step 2 or you need to make additional changes you should do that now, and commit the changes.

At this point your new-libs-repo is complete and you can push it to a server and make it live.

5) Now that the history for the libraries has been preserved in the new repo, it's time to force git to remove them from the original repository and have it re-write the history and commits.

If the repository is of any significant size removing the folders from the main repository will likely take some time. This process will cause all of the commit hashes to be rewritten. This command needs to be run for each folder that you are removing from the repository.

Back in the main software/ repository we want to checkout the master branch again, and remove all the files we just put into the new libraries repository.

    {% highlight bash %}
    cd ../software
    git checkout master
    git filter-branch --force --index-filter 'git rm -r --cached --ignore-unmatch lib/' --prune-empty --tag-name-filter cat -- --all
    git filter-branch --force --index-filter 'git rm -r --cached --ignore-unmatch docs/' --prune-empty --tag-name-filter cat -- --all
    {% endhighlight %}

Some unknown time later...

6) Even though git has finished filtering out the files you've removed, it keeps copies of the original commits, so your repository likely hasn't decreased in size, and it's likely it has increased in size. To clean out all the old commits do the following:

    {% highlight bash %}
    rm -rf .git/refs/original/
    git reflog expire --expire=now --all
    git gc --aggressive --prune=now
    {% endhighlight %}

7) Next create a new repository and pull just the master branch, as it is the only branch we've filtered.

    {% highlight bash %}
    cd ../
    mkdir new-software
    cd new-software
    git init
    git pull ../software master
    git gc
    {% endhighlight %}

At this point you can push this new repository to a fresh repository on your server.

At this point you should now have two repositories both with complete histories for all the files they contain.
