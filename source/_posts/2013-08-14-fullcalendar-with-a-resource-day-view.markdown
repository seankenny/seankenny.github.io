---
layout: post
title: "FullCalendar - with a resource day view!"
date: 2013-08-14 19:29
comments: true
categories: FullCalendar
---
> Update - this [Drag and Drop](http://www.seankenny.me/blog/2014/07/24/resource-fullcalendar-dragging-and-clicking/) post covers some other aspects of this plug-in.

[FullCalendar](http://arshaw.com/fullcalendar/) is a JavaScript calendar JQuery plug-in.  I've used it now on 2 projects and it's just great.

So on another project, we needed to be able to see the calendar agenda day view with it split into vertical columns for each 'resource'.  There are a few forks that are doing this with FullCalendar but they aren't really suitable for us.  One has resource rows rather than columns and the other is a fork of an old version of the FullCalendar code.

So I went ahead and forked the source on github.  The goal was to keep the forked version as similar to the core as possible - that way upstream merging should be easier.
<!--more-->
###Demo
[Here](http://embed.plnkr.co/8d16J15gKhE2IKCATspZ/preview) is a working demo of the plugin with resources.

###Code
The source code of it is [here](https://github.com/seankenny/fullcalendar).  Get the V2 js and css from here - [https://github.com/seankenny/fullcalendar/tree/v2/dist](https://github.com/seankenny/fullcalendar/tree/v2/dist).

###Usage
The only difference to the consumer of the plugin is that there is a new option ('resources') and there is a new property on the `eventObject` named `resourceId` that we use to tie back to the relevant resource.

```javascript
$('#calendar').fullCalendar({
  defaultView: 'resourceDay',
  resources: [{'id':'r1','name':'Resource 1'},{'id':'r2', 'name':'Resource 2'}],
  //resources: 'data-url'  //you can use just a url to your resources data if you want 
  events: [
  {
    title: 'R1-R2: Lunch 14.00-15.00',
    start: '2013-08-21T14:00:00.000Z',
    end: '2013-08-21T15:00:00.000Z',
    resources: ['r1','r2']
  }
});
```

The `event.resources` property can take any of the following formats:

```javascript
events: [
{
  resources: 'r1'  // to assign this event to the r1 resource
}
// OR
events: [
{
  resources: ['r1']  // to assign this event to the r1 resource
}
// OR
events: [
{
  resources: ['r1','r2']  // to assign this event to the r1 & r2 resources
}
```

The resources option on the calendar object also accepts a `className` to allow the styling of the individual resource columns:

```javascript
$('#calendar').fullCalendar({
  resources: [{'id':'r1', 'name':'Resource 1', 'className' : ['my-class-name']}]
```

Enjoy!