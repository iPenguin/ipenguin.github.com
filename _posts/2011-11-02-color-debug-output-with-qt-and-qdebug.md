---
title: "Color debug output with Qt4 and qDebug()"
layout: "post"
permalink: "/2011/11/color-debug-output-with-qt-and-qdebug.html"
uuid: "96765458961442037"
guid: "tag:blogger.com,1999:blog-3270817893928434685.post-96765458961442037"
date: "2011-11-02 19:14:00"
updated: "2011-11-02 19:16:29"
description: 
blogger:
    siteid: "3270817893928434685"
    postid: "96765458961442037"
    comments: "0"
categories: [output, debug, color, qt]
author: 
    name: "Brian C. Milco"
    url: "http://www.blogger.com/profile/05356031750889872461?rel=author"
    image: "http://img2.blogblog.com/img/b16-rounded.gif"
comments: true
---

I'm a very visual person, and color makes it a lot faster and easier for me to work. That's why I really like kDevelop with it's semantic highlighting. 
Anymore when I see grey text on a black console it feels blob-like and hard to read especially if the output is verbose.

I use Qt on a regular basis for my day job and while I love the framework it's debug output is vanilla. 
So I wanted to go from:

{% highlight bash %}
debug output
warning output
critical output
fatal output
{% endhighlight %}

to something a little more useful for finding problems:

<pre>
<span style="color: green;">MainWindow</span>::<span style="color: #0000aa;">MainWindow</span>(<span style="color: #6fa8dc;">QStringList</span>, <span style="color: #6fa8dc;">QWidget*</span>) : debug output
<span style="color: #f1c232;">Warning</span>: <span style="color: green;">MainWindow</span>::<span style="color: #0000aa;">MainWindow</span>(<span style="color: #6fa8dc;">QStringList</span>, <span style="color: #6fa8dc;">QWidget*</span>) : warning output
<span style="color: #cc0000;">Critical</span>: <span style="color: green;">MainWindow</span>::<span style="color: #0000aa;">MainWindow</span>(<span style="color: #6fa8dc;">QStringList</span>, <span style="color: #6fa8dc;">QWidget*</span>) : critical output
<span style="color: #cc0000;">Fatal</span>: <span style="color: green;">MainWindow</span>::<span style="color: #0000aa;">MainWindow</span>(<span style="color: #6fa8dc;">QStringList</span>, <span style="color: #6fa8dc;">QWidget*</span>) : fatal output
</pre>

Before we get started there are 2 main draw backs to this setup. 
First is that you can really only pass in QStrings. 
If you want to pass in anything else you have to wrap it like this:

{% highlight cpp %}
DEBUG("myNumber = " + QString::number(myNumber));
{% endhighlight %}

Second and more important kDevelop and QtCreator don't display the colorized output properly, 
and it actually makes it harder to read debug information. Keeping that in mind let's continue. 

With a little customization to Qt's message handler you can get a description and a little color:

errorhandler.h: 
{% highlight cpp %}
#ifndef ERRORHANDLER_H
#define ERRORHANDLER_H
void errorHandler(QtMsgType type, const char *msg)
{
    switch (type) {
        case QtDebugMsg:
            fprintf(stderr, "%s\n", msg);
            break;
        case QtWarningMsg:
            fprintf(stderr, "\033[1;33mWarning\033[0m: %s\n", msg);
            break;
        case QtCriticalMsg:
            fprintf(stderr, "\033[31mCritical\033[0m: %s\n", msg);
            break;
        case QtFatalMsg:
            fprintf(stderr, "\033[31mFatal\033[0m: %s\n", msg);
            abort();
    }
}
#endif // ERRORHANDLER_H
{% endhighlight %}

main.cpp:
{% highlight cpp %}
#include "errorhandler.h"
...
int main(int argc, char *argv[])
{
    qInstallMsgHandler(errorHandler);
    QApplication a(argc, argv);
    qDebug() << "debug output";
    qWarning() << "warning output";
    qCritical() << "critical output";
    qFatal("fatal output");
...
}
{% endhighlight %}

This will give you the following output:

<pre>
debug output
<span style="color: #f1c232;">Warning</span>: warning output
<span style="color: #cc0000;">Critical</span>: critical output
<span style="color: #cc0000;">Fatal</span>: fatal output
</pre>

That's a nice start but I also like to include the function name in the debug output so I know which class and function is giving me which output, but I hate having to add the information manually. 
What I want is `__PRETTY_FUNCTION__` or `Q_FUNC_INFO` (basically the same thing), but if you include that in the error handler above it's going to give you `void errorHandler(QtMsgType, const char*)`. 
Not very useful. So I put together some macros to pass messages and data to the actual message handlers with the `Q_FUNC_INFO` prepended.

debug.h:
{% highlight cpp %}
#ifndef DEBUG_H
#define DEBUG_H

#include <QDebug>

#define DEBUG(message) \
( \
    (qDebug() << Q_FUNC_INFO << ":" << QString(message).toStdString().c_str()), \
    (void)0 \
)

#define WARN(message) \
( \
    (qWarning() << Q_FUNC_INFO << ":" << QString(message).toStdString().c_str()), \
    (void)0 \
)

#define CRITICAL(message) \
( \
    (qCritical() << Q_FUNC_INFO << ":" << QString(message).toStdString().c_str()), \
    (void)0 \
)

#define FATAL(message) \
( \
    (qFatal("%s : %s", Q_FUNC_INFO, QString(message).toStdString().c_str())), \
    (void)0 \
)
#endif // DEBUG_H
{% endhighlight %}

Because of the formatting in the macros you can use them in code as if they were actual functions. 
Changing the debug lines in main() above to the following:

{% highlight cpp %}
DEBUG("debug output");
WARN("warning output");
CRITICAL("critical output");
FATAL("fatal output");
{% endhighlight %}

Results in the output:

<pre>
MainWindow::MainWindow(QStringList, QWidget*) : debug output
<span style="color: #f1c232;">Warning</span>: MainWindow::MainWindow(QStringList, QWidget*) : warning output
<span style="color: #cc0000;">Critical</span>: MainWindow::MainWindow(QStringList, QWidget*) : critical output 
<span style="color: #cc0000;">Fatal</span>: MainWindow::MainWindow(QStringList, QWidget*) : fatal output
</pre>

Useful but still a lot of blob-like black text. 
So I decided to create a function to parse the `Q_FUNC_INFO` string and add a little more color. 
So I added this to the debug.cpp files.

debug.cpp:
{% highlight cpp %}
#include "debug.h"
#include <QStringList>
#include <QString>

QString colorizeFunc(QString name)
{
    QString output;
    QStringList classParts = name.split("::");
    QStringList nameAndType = classParts.first().split(" ");

    QString returnType = "";
    if(nameAndType.count() > 1)
        returnType = nameAndType.first() + " ";
    QString className = nameAndType.last();

    QStringList funcAndParamas = classParts.last().split("(");
    funcAndParamas.last().chop(1);
    QString functionName = funcAndParamas.first();
    QStringList params = funcAndParamas.last().split(",");

    output.append("\033[036m");
    output.append(returnType);
    output.append("\033[0m\033[32m");
    output.append(className);
    output.append("\033[0m::");
    output.append("\033[34m");
    output.append(functionName);
    output.append("\033[0m(");

    QStringList::const_iterator param;
    for (param = params.begin(); param != params.constEnd(); ++param) {
        if(param != params.begin()) {
            output.append("\033[0m,");
        }
        output.append("\033[036m");
        output.append((*param));
    }
    output.append("\033[0m)");
    return output;

}
{% endhighlight %}

Then in the macros replace `Q_FUNC_INFO` with `debugFunctionName(Q_FUNC_INFO).toStdString().c_str()` and the output will look like this:

<pre>
<span style="color: green;">MainWindow</span>::<span style="color: #0000aa;">MainWindow</span>(<span style="color: #6fa8dc;">QStringList</span>, <span style="color: #6fa8dc;">QWidget*</span>) : debug output
<span style="color: #f1c232;">Warning</span>: <span style="color: green;">MainWindow</span>::<span style="color: #0000aa;">MainWindow</span>(<span style="color: #6fa8dc;">QStringList</span>, <span style="color: #6fa8dc;">QWidget*</span>) : warning output
<span style="color: #cc0000;">Critical</span>: <span style="color: green;">MainWindow</span>::<span style="color: #0000aa;">MainWindow</span>(<span style="color: #6fa8dc;">QStringList</span>, <span style="color: #6fa8dc;">QWidget*</span>) : critical output
<span style="color: #cc0000;">Fatal</span>: <span style="color: green;">MainWindow</span>::<span style="color: #0000aa;">MainWindow</span>(<span style="color: #6fa8dc;">QStringList</span>, <span style="color: #6fa8dc;">QWidget*</span>) : fatal output
</pre>