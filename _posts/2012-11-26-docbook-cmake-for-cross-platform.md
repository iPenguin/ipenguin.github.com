---
title: "Docbook & CMake for cross platform documentation"
permalink: "/2012/11/docbook-cmake-for-cross-platform.html"
uuid: "5201424067889033394"
guid: "tag:blogger.com,1999:blog-3270817893928434685.post-5201424067889033394"
date: "2012-11-26 17:00:00"
updated: "2012-11-26 17:00:08"
excert: "Using cmake to generate docbooks files into platform specific documentation"
blogger:
    siteid: "3270817893928434685"
    postid: "5201424067889033394"
    comments: "0"
header:
    teaser: '/assets/images/cmake-logo-sm.png'
categories: [documentation, docbook, pdf, cmake, cross platform]
comments: true
---

Docbook XML is a great technology but it takes some work to get it all setup.
This post will hopefully jump start the process for you.

I've put together a cmake script (DocbookGen.cmake) that makes it a lot easier to convert the documentation from Docbook markup to one of several common formats: plain html, pdf, and Microsoft html help (chm files),
and I put the files in [a gist](https://gist.github.com/4142447).

What you'll need:

For *buntu:
`sudo apt-get install docbook-xsl-ns xsltproc fop cmake`


  [docbook-xsl-ns](http://sourceforge.net/projects/docbook/files/docbook-xsl-ns/)  
  xsltproc (native on Mac and linux, use [cygwin installer](http://cygwin.com/install.html) on Windows)  
  [fop](http://xmlgraphics.apache.org/fop/download.html)  
  [cmake](http://cmake.org/cmake/resources/software.html)  
  [The official docbook guide](http://docbook.org/tdg51/en/html/) (for syntax reference)  
  [hhc](http://msdn.microsoft.com/en-us/library/windows/desktop/ms669985(v=vs.85).aspx) Microsoft HTML Help SDK for Windows.

Project structure:

    [PROJECT]/
        cmake/modules/
             DocbookGen.cmake
        docs/
            index.docbook.in
            myStyle.xsl.in
            images/cover.png

You should save the DocbookGen.cmake module as described in the project structure.
You may need to edit paths in the module depending on your installations.

DocbookGen.cmake:

{% gist iPenguin/4142447 DocbookGen.cmake %}

I've included a basic docbook file (index.docbook.in) with a simple book structure for creating documentation.
If you are using the myStyle.xsl.in file unaltered you'll also need to create a cover image for the pdf file.

index.docbook.in:

{% gist iPenguin/4142447 index.docbook.in %}

By using configure_file with index.docbook.in you can use variables in cmake to generate the common values that are needed in the documentation as well as the package and software.
Docbook doesn't have the ability to do conditional imports which is an issue because you have to import the default xsl files on each platform and they usually have different paths (especially on Windows).
To get around this issue I used CMake's ability to configure files to specify which path to use based on the platform.
You can see this on line 9 in the myStyle.xsl.in file below.

myStyle.xsl.in:

{% gist iPenguin/4142447 myStyle.xsl.in %}

Use the code in the CMakeLists.txt file to create the documentation files you need.

CMakeLists.txt:

{% gist iPenguin/4142447 CMakeLists.txt %}

Once you've got the documentation built you'll have to "install" it into the package in the CMakeLists.txt file:

{% highlight cmake %}
install(FILES "${CMAKE_BINARY_DIR}/docs/pdf/${PROJECT_NAME}_User_Guide_${VERSION_SHORT}.pdf" DESTINATION bin)
{% endhighlight %}

Once you've got the process down you just need to write your documentation, docbook syntax is verbose, especially if you're used to writing HTML or markdown.
