---
layout: post
title: "Web Api with dot in the route"
date: 2014-06-04 17:58
comments: true
categories: [ASP.Net, Web Api]
keywords: aweb api routes, named routes, special characters
description: How to have a dot/period in a web api route
---
We have a requirement to have a dot midway in a route for one of our web api services.  The route looks something like `http://localhost/api/some.route/person`.  The controller (if using attribute based routing) is:
 
```c#
[RoutePrefix("api/some.route")]
public class UsersController : ApiController
{
  [Route("person")]
  public HttpResponseMessage Person(int id)
  {
    // do stuff
  }
}
```
 <!--more-->
When you try to hit this resource using fiddler or postman, you'll get a 404 not found error.  There are plenty of StackOverflow posts out there about having dots in the route but these mainly are to do with a dot at the end of the route rather than midway so something like `http://localhost/api/someroute/per.son`.  IIS will identify the dot and assume you are after a static file with an extension of 'son'.  To get around this, you can add this to your web.config file as explained by [Phil Haack](http://haacked.com/archive/2010/04/29/allowing-reserved-filenames-in-URLs.aspx).
 
```xml
<system.web>
  <httpRuntime relaxedUrlToFileSystemMapping="true" />
```
 
However this will not work where the dot is midway in the URL like I have.  For me, adding the following resolves the issue (although not having to have a dot in the route would be the obvious solution but sometimes your hands are tied).
 
```xml
<system.webServer>
  <handlers>
    <add name="UrlRoutingHandler" type="System.Web.Routing.UrlRoutingHandler, System.Web, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" path="some.route/*" verb="*"/>
```
 
Adding this post as I **KNOW** I will forget this!