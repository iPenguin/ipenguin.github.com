---
title: "Writing Git Hooks in Node.js"
permalink: "/2016/11/writing-git-hooks-in-nodejs.html"
date: "2016-11-30 17:30:00"
updated: "2016-11-30 17:30:00"
excerpt: "A quick tutorial on writing git hooks using javascript and Node.js"
header:
  teaser: "/assets/images/nodejs-sm.png"
categories: [Node.js, git, hooks, readline, commander]
comments: true
---

At work we use the post-receive git hook to automatically deploy websites on a number of our servers. Among a few other things the hook is used to recreate cached pages when templates are modified. Lately, we've added some additional tasks to the post-receive hook and it is becoming more complicated. So when we wanted to add automatic minification of the CSS and JavaScript resources to the hook, we made the decision to convert it to Node.js.

We choose Node.js because we were planning to use two minifiers that required Node.js anyway, and we are looking into the possibility of using Node.js for future projects, so this hook was a good, low risk, place to work with Node.js and see how it will fit in at our business.

### Getting Data from Git

If you've written git hooks before you know that any information passed to the hook is passed via stdin and not as parameters to the script. To get the data that git passes to the hook you need to include and use the readline node module.

Install readline and save it with your git hook project:
{% highlight bash %}
 npm install --save readline
{% endhighlight %}

Include readline and parse the data from stdin:
{% highlight javascript %}
 const readline = require( 'readline' );
 const rl = readline.createInterface( {
     input:  process.stdin,
     output: process.stdout,
 } );

 rl.on( 'line', ( input ) => {
     let data = input.split( ' ' );
     main( data[ 0 ], data[ 1 ], data[ 2 ] );
 } );

function main( oldRev, newRev, branch ) {
    //Do minification, caching and other fun stuff.
    ...
}
{% endhighlight %}

### Testing your Hook on the Command Line

I prefer to test my scripts on the command line. I also find it helpful to be able to run the deployment process manually should something fail. You can make it possible to run the hook on the command line by accepting command line parameters in your Node.js script. You can do this by adding and requiring commander.

{% highlight bash %}
 npm install --save commander
{% endhighlight %}

{% highlight javascript %}
const program = require( 'commander' );

program
    .description( 'git post-receive hook' )
    .usage( '[options]' )
    .option( '-b, --branch <branch>', 'Branch of the repo being committed' )
    .option( '-n, --new <newRevision>', 'New revision we would like to deploy' )
    .option( '-o, --old <oldRevision>', 'Old revision currently deployed' )
    .parse( process.argv );

// if we've set any flags we're running the script command line...
if( program.old || program.new || progarm.branch ) {
    main( process.old, process.new, process.branch );
    process.exit();
}

// otherwise we're running the script as a hook...
rl.on( 'line', ( input ) => {
    console.log( "running as hook" );
    let data = input.split( ' ' );
    main( data[ 0 ], data[ 1 ], data[ 2 ] );
} );

process.exit();

function main( oldRev, newRev, branch ) {
    ...
}
{% endhighlight %}

When I started this project I took inspiration from [Git Hooks for the Front End Developer](http://blog.bradleygore.com/2015/08/07/git-hooks-for-the-front-end-developer/) by Bradly Gore.
