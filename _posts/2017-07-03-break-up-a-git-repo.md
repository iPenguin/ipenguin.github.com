---
title: "Break Up a Git Repository"
permalink: "/2017/06/break-up-a-git-repository.html"
date: "2017-06-03 20:15:01"
updated: "2017-06-03 20:15:01"
excerpt: "A quick reference for separating out a git repository while preserving the history."
header:
  teaser: "/assets/images/git-icon.png"
categories: [git, repository, tips, maintenance ]
comments: true
---

There are times when you need to break your code base into smaller more maintainable pieces. For example when you decide to make a shared library out of some of your code. Or that handful of images has turned into an entire catalog.

You could just move the library folder you want out of the main project and start a new git repository, but you would loose the entire history of that folder, making it harder to find and fix bugs, and keep track of changes.

### Notes before we begin

 * Once this process is complete *all* the hashes will be re-written, and anyone with a copy of the repository will have to delete there copy and clone the new repositories. If they have their own branches they're working on they'll need to create patches and apply them against the new repositories.

 * Because you have to delete the existing repository and rewrite the hashes you'll need to find a time when you can do the transition that does not interfere with your teams development work, or schedule it.

 * Most of the process goes quickly but the filter-branch step can take awhile on large repositories, so I'd recommend doing at least one dry run to figure out how much time you'll need to complete the process for your project.

### What we're going to be doing

The process will generally be as follows:

1. Clone the primary repository
2. Create a branch, and using git remove all files not in the libs/ directory.
3. Reorganize any files in the library structure.
4. Create a new empty folder and repository for the new library repository.
5. Go back to the master branch and repeat steps 2-5 for the all other directories.
6. In the new folder pull the branch you just created.



### How to do it

For example if you have a git repository with the following structure:

/software/libs/
/software/app/

and you want to break out the libraries into a separate git repo to make them easier to share between some other projects.

First:

{% highlight bash %}
 git clone https://github.com/user/software.git
 cd software/
{% endhighlight %}

To split the repository and keep the files in the /libs directory do the following...

{% highlight bash %}
 git checkout -b shared-libs
 git filter-branch --prune-empty --subdirectory-filter /libs
{% endhighlight %}

If you want to reorganize the files this is the time to do it. Make your changes and commit them.

cd ../
mkdir split-a-repo
cd split-a-repo
git init

-- end repeat --

git pull ../main-repo split-a
git pull ../main-repo split-b --allow-unrelated-histories

remove folders from main-repo:

-- repeat for each folder to remove: --
git filter-branch --force --index-filter 'git rm -r --cached --ignore-unmatch folder-a' --prune-empty --tag-name-filter cat -- --all
-- end repeat --

rm -rf .git/refs/original/
git reflog expire --expire=now --all
git gc --aggressive --prune=now
`
then create a new repo and pull just the master branch:

cd ../
mkdir temp.repo
cd temp.repo
git init
git pull ../main-repo master
