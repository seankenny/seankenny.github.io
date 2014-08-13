---
layout: post
title: "FullCalendar - With a Resource Day View! - v2-0-2"
date: 2014-08-11 18:52:44 +0100
comments: true
categories: [FullCalendar]
---
###v2.0.2 `beta!`
I've finally got to updating my fork of Adam Shaw's v2.0.2 <a href="http://arshaw.com/fullcalendar/" target="_blank">Full Calendar</a>.  

There are a few changes required if you are going to move from the previous 1.6 version, namely the use of the <a href="http://momentjs.com/" target="_blank">moment.js</a> date library.  I've used this library previously and you owe it to yourself to take a look.  It takes all that javascript date pain away.  And that's a good thing...
<!--more-->
The ResourceDayView I've added is pretty much a clone of the AgendaDayView so head on over to <a href="http://arshaw.com/fullcalendar/docs/" target="_blank">arshaw.com/fullcalendar/docs</a> to get a feel for the agenda view.

The <a href="https://github.com/arshaw/fullcalendar/wiki/Upgrading-to-v2" target="_blank">upgrade to v2</a> docs cover the diffs in detail.

###Demo
Take a look at the plunkr demo <a href="http://embed.plnkr.co/cX9dH8eTjKaddJ0Gpw21/preview" target="_blank">here</a>.

In the demo, pop open the developer tools (F12 in chrome) => console and you should see a console log for clicking on events, dragging events, etc.

###Issues
If you spot any issues, please submit a <a href="https://github.com/seankenny/fullcalendar/issues" target="_blank">github issue</a> with a label of v2.0!

###v2.1
I see there is a v2.1 release imminent &#42;*le sigh*&#42;.  It is still in beta and there are a SHED LOAD of internal changes so as soon as it is released I'll start the merge process again.

###Related posts
>[FullCalendar - With a Resource Day View!](http://www.seankenny.me/blog/2013/08/14/fullcalendar-with-a-resource-day-view/)

>[Resource FullCalendar - Dragging and Clicking](http://www.seankenny.me/blog/2014/07/24/resource-fullcalendar-dragging-and-clicking/)