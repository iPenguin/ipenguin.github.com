---
title: "CMake: Automatically use git tags as version strings"
layout: "post"
permalink: "/2012/11/cmake-automatically-use-git-tags-as.html"
uuid: "271260801997189329"
guid: "tag:blogger.com,1999:blog-3270817893928434685.post-271260801997189329"
date: "2012-11-25 00:38:00"
updated: "2012-11-25 01:32:42"
description: 
blogger:
    siteid: "3270817893928434685"
    postid: "271260801997189329"
    comments: "0"
categories: [c++, version, git, revisions, cmake]
author: 
    name: "Brian C. Milco"
    url: "http://www.blogger.com/profile/05356031750889872461?rel=author"
    image: "http://img2.blogblog.com/img/b16-rounded.gif"
comments: true
---

If you're like me and you use git to tag your new versions, you'd like your build system to automatically include those git tags when you build the software. 
Depending on the complexity of your project, the following instructions could make releasing a new version of your software as simple as:

{% highlight bash %}
$ git tag -a "v1.5.0"
$ cd build/
$ cmake ../
$ make
$ cpack
{% endhighlight %}

Note I'm using Qt4 in the sample below, if you change the `QString`s to `std::string`s it should still work.

First you'll need a way to get the current git tag and description information into cmake. 
So you'll want to use the scripts GetGitRevisionDescription.cmake and GetGitRevisionDescription.cmake.in from [https://github.com/rpavlik/cmake-modules](https://github.com/rpavlik/cmake-modules). 
Add these files to your project. I like to include them in a `[PROJECT]/cmake/modules` folder.

My projects usually look something like this:

    [Project]/
        build/
        cmake/
            modules/
        src/
        CMakeLists.txt
        ...

In your CMakeLists.txt file include the cmake/modules folder in your modules path as follows:

{% highlight cmake %}
# Appends the cmake/modules path to MAKE_MODULE_PATH variable.
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules ${CMAKE_MODULE_PATH})
{% endhighlight %}

Now that you can get the version information in your cmake files you need to parse it.

My version strings look like this: `v1.4.0` or `v1.4.1-7-ge3b1138-d` with all the git commit information, so first I like to parse them and get the major, minor, patch, and sha1 pieces. 
For most uses I only need the first three pieces, so I have added a `VERSION_SHORT` variable that combines them for convenience.
I can use these variables through out the build process, in the cpack packaging process, and pass them into the source code so I can use it in the software as well. 

CMakeLists.txt:
{% highlight cmake %}
# Add the following lines to your CMakeLists.txt file
#
# Make a version file containing the current version from git.
#
include(GetGitRevisionDescription)
git_describe(VERSION --tags --dirty=-d)

#parse the version information into pieces.
string(REGEX REPLACE "^v([0-9]+)\\..*" "\\1" VERSION_MAJOR "${VERSION}")
string(REGEX REPLACE "^v[0-9]+\\.([0-9]+).*" "\\1" VERSION_MINOR "${VERSION}")
string(REGEX REPLACE "^v[0-9]+\\.[0-9]+\\.([0-9]+).*" "\\1" VERSION_PATCH "${VERSION}")
string(REGEX REPLACE "^v[0-9]+\\.[0-9]+\\.[0-9]+(.*)" "\\1" VERSION_SHA1 "${VERSION}")
set(VERSION_SHORT "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules/version.cpp.in
                ${CMAKE_CURRENT_BINARY_DIR}/version.cpp)
set(version_file "${CMAKE_CURRENT_BINARY_DIR}/version.cpp")

#Add the version_file to the project build or it won't compile.
add_executable(${PROJECT_NAME} ${source_files} ${ui_files} ${version_file})

{% endhighlight %}

As you can see above the configure_file line configures a source file that only contains the version strings. 
This source file is in two parts.
The version.h file should be in the source directory of your project so you can include it in your project and work with the version strings. 

version.h:

{% highlight cpp %}
#ifndef VERSION_H
#define VERSION_H

#include <QString>

//Global version strings
extern const QString gVERSION;
extern const QString gVERSION_SHORT;

#endif //VERSION_H
{% endhighlight %}

The second part is the `version.cpp.in` file that I've stored with the other files in cmake/modules. 
The configure_file command in cmake automatically replaces variables that start and end with the "@" symbol. 
In this case, when cmake is run it will automatically replace the `@VERSION@` and `@VERSION_SHORT@` with the correct version strings.

version.cpp.in:

{% highlight cpp %}
//the path to the version.h file is relative to your build directory.
#include "../src/version.h"

const QString gVERSION = "@VERSION@";
const QString gVERSION_SHORT = "@VERSION_SHORT@";
{% endhighlight %}

With all these pieces you should be able to use the `gVERSION` and `gVERSION_SHORT` anywhere you `#include "version.h"`.
You can also use the `${VERSION}` and `${VERSION_SHORT}` anywhere in your cmake build files including in cpack which means updating your installers and packages is a snap too.

UPDATE: Apparently there have been changes made to the GetGitRevisionDescription.cmake file. If you edit it and remove line 110: `${hash}` it should work.

I've posted a working copy on github: [https://github.com/iPenguin/version_git](https://github.com/iPenguin/version_git)
To try it:

{% highlight bash %}
$ git clone https://github.com/iPenguin/version_git.git

$ cd version_git

$ mkdir build && cd build

$ cmake .. && make

$ ./version_from_git
{% endhighlight %}
