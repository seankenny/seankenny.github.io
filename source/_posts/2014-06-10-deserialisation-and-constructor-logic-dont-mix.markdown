---
layout: post
title: "Deserialisation and constructor logic don't mix"
date: 2014-06-10 20:11
comments: true
categories: [ASP.Net, Web Api]
keywords: c#, deserialization performance, json, xml
description: Avoiding deserialization performance hots by avoiding expensive constructor logic
---
I got caught on this one last week. I was load testing a web api resource using ApacheBench and was getting some really bad perf figures in the region of 40 req/sec when I would have expected an order of magnitude more at least.  The data model being sent as JSON was:
<!--more-->
```c#
public class LogMessage
{
        public ulong PhysicalMemory { get; set; }
        // lots more properties
       
        public LogMessage()
        {
                  var ci = new ComputerInfo();
                  AvailablePhysicalMemory = ci.AvailablePhysicalMemory;
        }
}
```
When deserialising the request body into the dataModel, the constructor gets called and we have the overhead of the **slow** `ComputerInfo().AvailablePhysicalMemory` call to get the available memory of the machine.  Multiply this in a load test scenario and there's the culprit.
 
The fix is simple:
 
```c#
public class LogMessage
{
        public ulong AvailablePhysicalMemory { get; set; }
       
        public void Init()
        {
                  var ci = new ComputerInfo();
                  AvailablePhysicalMemory = ci.AvailablePhysicalMemory;
        }
}
```
 
The client changes to:
 
```c#
var message = new LogMessage().Init();
```