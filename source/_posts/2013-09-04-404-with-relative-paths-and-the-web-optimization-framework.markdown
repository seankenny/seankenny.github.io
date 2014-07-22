---
layout: post
title: "404 with relative paths and the Web Optimization Framework"
date: 2013-09-04 18:54
comments: true
categories: [ASP.Net, MVC, Web Optimization Framework, Minification, CSS]
---
We want to minify and combine our javascript and css files when moving into a production environment to reduce as much as we can the time taken for our web pages to load.  In .Net land, we might use the [Microsoft ASP.NET Web Optimization Framework](http://www.nuget.org/packages/microsoft.aspnet.web.optimization/) to do this.

There is sometimes an issue where a css file has a relative path reference to an image.  An example of this is in twitter bootstrap.  See that url?  That's a problem for us.
<!--more-->
```css
[class^="icon-"],
[class*=" icon-"] {
  display: inline-block;
  width: 14px;
  height: 14px;
  margin-top: 1px;
  *margin-right: .3em;
  line-height: 14px;
  vertical-align: text-top;
  background-image: url("../img/glyphicons-halflings.png");
  background-position: 14px 14px;
  background-repeat: no-repeat;
}
```

We would create a bundle:

```c#
bundles.Add(new StyleBundle("~/styles/css").Include(
                "~/Content/bootstrap/css/bootstrap.css",
                "~/Content/bootstrap/css/bootstrap-responsive.css",
                "~/Content/site.css"));
```

When we minify this and all our site specific stylesheets into a bundle and then render the HTML, we'll get a `404 HTTP NOT FOUND` exception that we can see in the chrome developer tools (under the `network` tab).  The bundle will be generated as 'styles/css/' and the url will check for the image in `styles/css/../img/glyphicons-halflings.png`.

Luckily there is an option in the optimization framework called `CssRewriteUrlTransform()` to handle this:

```c#
bundles.Add(new StyleBundle("~/styles/app").Include(
                "~/Content/bootstrap/css/bootstrap.css", new CssRewriteUrlTransform()).Include(
                "~/Content/bootstrap/css/bootstrap-responsive.css",
                "~/Content/site.css"));
```

No more 404.