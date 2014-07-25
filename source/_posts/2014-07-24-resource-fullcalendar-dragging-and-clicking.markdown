---
layout: post
title: "Resource FullCalendar - dragging and clicking"
date: 2014-07-24 17:31:05 +0100
comments: true
categories: FullCalendar
---
I've previously posted on my "addition of resources" fork of the fullcalendar javascript control [here](http://www.seankenny.me/blog/2013/08/14/fullcalendar-with-a-resource-day-view/).  Today I want to outline some features around the drag/drop and click events that are a little undiscoverable.

###Select
To add a select event callback, use the following config setting:

```javascript
    ....
    selectable: true,
    select: function(start, end, allDay, ev) {
        console.log(start);
        console.log(end);
        console.log(ev.data);
    },
    ....
```
<!--more-->

This is triggered when the user clicks (or selects) the calendar (i.e. not on a calendar event).

The start and end date of the the area you click (or click, drag and released) will be available in the first 2 params.  The `ev` parameter will contain an `ev.data` property which will contain the resource you clicked on.  In the above example, you would see something like this in chrome's dev tools console window:

```
    Thu Jul 24 2014 06:00:00 GMT+0100 (GMT Summer Time) 
    Thu Jul 24 2014 09:30:00 GMT+0100 (GMT Summer Time) 
    Object {id: "resource1", name: "Resource 1", className: Array[0]}
```

Do not confuse `ev` with a calendar `event`!

###Calendar Event clicking
To add a calendar event click callback, use the following:

```javascript
    ....
    eventClick: function(event) {
        console.log(event.resources);
    },
    ....
```

This one is the same as the original fork with the addition of `resources` which provides the resource id(s) of the calendar event:
```
    ["resourceid1","resourceid2"]
```

###Calendar Event drag and drop
```javascript
    ....
    eventDrop: function (event, delta, revertFunc) {
          console.log(event.resources);
    },
    ....
```
```
    ["resourceid1","resourceid2"]
```

###Confusing?
The resource data available on these are not consistant.  For `eventClick` and `eventDrop` the calendar event is returned and this object has the `resources` array on it with the resource ids.  This is as you would set up the calendar initially.

The `select` has no event associated with it yet - you only get the drag start and end date/times.  This is Ok until you need to figure out the resource you are interested in.  The click event has a data property which seems to fit the bill.  Maybe I should add a `resources` array for the id's instead?  Would lead to:

```javascript
    select: function(start, end, allDay, ev, resources) {
        console.log(resources);
    }
```
```
    ["resourceid1"]
```

All of the above can be seen in action on this [plunkr](http://embed.plnkr.co/8d16J15gKhE2IKCATspZ/preview)