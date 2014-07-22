---
layout: post
title: "Web Api, Entity Framework and ISO 8601 date formats"
date: 2013-09-19 17:50
comments: true
categories: [Entity Framework, Web Api, ISO 8601]
---
For those using Web Api to deliver json payloads, you might come across an issue relating to dates, more specifically, timezones.

The `ISO8601` date format is becoming the defacto format for date strings and is now the default for the default Web Api serialiser, [Json.Net](http://james.newtonking.com/pages/json-net.aspx).

The `ISO8601` date specification explicitly states what happens when the `TZD` (or time zone designator) is ommited.  Read more [here](http://en.wikipedia.org/wiki/ISO_8601#Time_zone_designators). It states:
<!--more-->
> If no UTC relation information is given with a time representation, the time is assumed to be in local time.

###Entity Framework
We should always store datetime as UTC in the database.  However, when we are retrieving an Entity Framework model, date properties are given a `DateTime.DateKind` of `Unspecified`.  When the Json.Net serialiser serialises this, it has to assume that the datetime is not UTC and so the date will serialise to a string as `2013-09-19T07:00:00` rather than `2013-09-19T07:00:00Z` <-- see the 'Z' bit? That's the `TZD` UTC indicator.  This is a fair call from Json.Net.

There is no easy way to get Entity Framework (Code First) to set the DateKind to UTC without either reflecting through each and every property when a model is retrieved by the `DbContext` (by subscribing to the `ObjectContext.ObjectMaterialized` event) or by adopting a more manual approach using:

```c#
DateTime.SpecifyKind(myDate, DateTimeKind.Utc);
```

on each datetime property.

###EcmaScript 5
Now onto another (related) issue.  For some reason I cannot fathom, the ECMAScript 5 ISO8601 specification differs fundamentally from the [ISO spec](http://es5.github.io/#x15.9.1.15):

> The value of an absent time zone offset is “Z”.

So those implementing to the ISO spec will handle the date as 7am local time whereas those implementing to ECMAScript 5 spec will handle as 7am UTC.  Not nice.

To handle this, we need to always pass UTC dates from our web Api and, because we are doing this, we can enforce the serialiser to append the UTC `TZD` of `Z` by adding this to the Web Api configuration:

```c#
 var jsonFormatter = GlobalConfiguration.Configuration.Formatters.JsonFormatter;
 jsonFormatter.SerializerSettings.DateTimeZoneHandling = DateTimeZoneHandling.Utc;
```

It's a pretty blunt approach but it works p