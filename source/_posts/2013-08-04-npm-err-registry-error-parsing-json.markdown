---
layout: post
title: "NPM - ERR! registry error parsing json"
date: 2013-08-04 14:44
comments: true
categories: node npm
---
Today I upgraded to node 0.10.15 and just ran npm install on a project I am working on.  I started seeing a lot of these failures:

    npm - ERR! registry error parsing json
    npm - ERR! SyntaxError: Unexpected token <

I had already run the npm install command maybe 30 minutes previously so I thought it might be something to do with the node/npm versions so I did the usual and rolled back.  Same issue.  Ok so maybe a corrupt module or something?  `npm cache clean` was next.  Still no change.
<!--more-->
Ok so what the heck is going on?  Looking a bit more closely showed that npm appeared to be setting the mime type to html on the reponse.  Hang on - the npm registry does not return html.  So somewhere along the line a proxy or some other intermediary was setting the type.

Anyway, long story short - reboot of the laptop and we were back in business.  I can only guess that, with the various upgrade installs and some other tasks I was running, something caused this mess.

Just one of those things.