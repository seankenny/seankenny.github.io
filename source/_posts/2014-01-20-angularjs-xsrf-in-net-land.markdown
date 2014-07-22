---
layout: post
title: "AngularJS XSRF in .Net land"
date: 2014-01-20 18:31
comments: true
categories: [.net, angularjs, web api, authentication]
---
Ah yes, [XSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery).

So the basics are we need to ensure that the content of a POST request is coming from our site and has not been intercepted by a ne'er-do-well. 

In ASP.Net MVC it's pretty straighforward.  In your Razor file, just add an `Html.AntiForgeryToken()` into the form in question and then, on the action, add a `[ValidateAntiForgeryToken]` filter and all is well.
<!--more-->
For angularjs and web api, things are different.  First off, we are using HTML files and not razor cshtml files so we cannot use the `Hml.AntiForgeryToken()` helper method.

So this is the way I have done it using the FormsAuthentication cookie (kudos to this [SO](http://stackoverflow.com/questions/15574486/angular-against-asp-net-webapi-implement-csrf-on-the-server) post.  Check it out - all you need is in there.  Remember to up-vote!).  

1)  Add a `XSRF-TOKEN` cookie.  The value of this cookie is obtained from the authorization cookie.
AngularJs, by default, will search for this cookie when POSTing data back to the server.  When it finds it, it will add a `X-XSRF-TOKEN` header to the request.

2) When we receive a POST request, we add a `[ValidateXSRFToken]` filter which will look for the `X-XSRF-TOKEN` request header then compare it to the authentication cookie value to see if there is a match.

If there is no match, we return a `401 Unauthorized` response.

Now for us, we cannot use a cookie names `XSRF-TOKEN` for reasons I won't go into.  So we can alter this in the server side code easily and then set the angularjs code to use this name rather than the default `XSRF-TOKEN` name.  we do this in the angular.module().config section of the code:

```js
angular.module('myApp', [])
    .config(['$httpProvider', function ($httpProvider) {
        $httpProvider.defaults.xsrfCookieName = 'FORM-XSRF';
    });
```

Now angular will tie up nicely to the XSRF cookie change.

If you need to change the POST request header from `X-XSRF-TOKEN` you can use:
```js
$httpProvider.defaults.xsrfHeaderName  = 'X-FORM-XSRF';
```

Next, what happens when you [introduce `slidingExpiration="true"` into the mix](http://seankenny.me/blog/2014/01/22/angularjs-xsrf-in-net-land-slidingexpiration/)?