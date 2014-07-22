---
layout: post
title: "CamelCaseJson"
date: 2014-01-05 10:49
comments: true
categories: [.net, angularjs, web api]
---
I prefer JSON camel cased.  The default setup in ASP.net Web Api using the Newtonsoft JsonSerializer is for Pascal Case.  To switch to camel case is easy and makes for a much more seamless angularJS and Web Api interaction.  In the WebAPiConfig class:

```c#
var jsonFormatter = GlobalConfiguration.Configuration.Formatters.JsonFormatter;
jsonFormatter.SerializerSettings.ContractResolver = new CamelCasePropertyNamesContractResolver();
```