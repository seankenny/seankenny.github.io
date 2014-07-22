---
layout: post
title: "AngularJs in HTML5 Mode with Express"
date: 2013-08-05 18:49
comments: true
categories: angularjs expressjs html5
---
By default, the routing in angular utilises the hash character in the URI path.  For example `http://myapp/#/customers/5`. By setting the $locationProvider to html5Mode, we can get to `http://myapp/customers/5` which is a lot nicer.

```javascript
angular.module('myApp', []).
  config(['$routeProvider', '$locationProvider', function($routeProvider, $locationProvider) {
  $routeProvider.
      when('/customers/:customerId', {templateUrl: 'partials/customer.html', controller: CustomerCtrl}).
      otherwise({redirectTo: '/'});

  $locationProvider.html5Mode(true);

}]);
```
<!--more-->
However, when the page is refreshed, the browser has no way of knowing it should hit `http://myapp/` and then defer control to angular's routing.  Instead, it will do a `GET` request to the `http://myapp/customers/5` endpoint.  Which of course, does not exist.

To allow for this, we need to have a catch-all route in our web server, [Express](http://expressjs.com/) for this example.

```javascript
var app = require('express')();

app.configure(function(){
  // static - all our js, css, images, etc go into the assets path
  app.use('/assets', express.static('/assets'));
  //If we get here then the request for a static file is invalid so we may as well stop here
  app.use('/assets', function(req, res, next) {
    res.send(404); 
  });

  app.get('/customers/:id', function(req, res){
    // return data for customer....
  });

  // This route deals enables HTML5Mode by forwarding missing files to the index.html
  app.all('/*', function(req, res) {
    res.sendfile('index.html');
  });
});

app.listen(9000);
```