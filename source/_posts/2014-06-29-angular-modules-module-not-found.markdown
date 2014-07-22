---
layout: post
title: "Angular Modules - module not found"
date: 2014-06-27 19:42
comments: true
categories: [angularJS]
keywords: angularjs modules, missing controller, remove global app variable
description: How to fix a missing controller if using angularjs modules to load
---
This little one caught me out for an hour or two.  Moving away from the global 'app' variable in angularJs, I started using the angular.module syntax:

```javascript
angular.module('UserHandler', [])
```
<!--more-->
The app.js starts things using an IIFE [(Immediately Invoked Function Expression)](http://benalman.com/news/2010/11/immediately-invoked-function-expression/): 

```javascript
(function() {
  'use strict';

  var app = angular.module('bootstrapper', ['ngRoute']);

  // do other config things with the app like routing, etc.
)();
```
 
 Then onto my controllers:
 
```javascript
(function(app) {
  'use strict';

  app.controller('registrationController', [
    '$scope', function($scope) {
    // registration stuff
    }
  ]);
})(angular.module('bootstrapper', []));  // this is bad
```

```javascript
(function(app) {
  'use strict';

  app.controller('signinController', [
    '$scope', function($scope) {
    // sign in stuff
    }
  ]);
})(angular.module('bootstrapper', []));  // this is bad
```
 
 When I run this, the ```registrationController``` is never found.  Moving it to be declared after the ```signinController``` leads to the ```signinController``` never being found.  Huh?
 
The secret is in the array on the ```angular.module()``` call.  Adding the array tells angular that you want a new angular module which dumps the initial one that you just created for the first controller.
 
 The correct way to create the controllers using this form of controller set up is:
 
```javascript
(function(app) {
  'use strict';

  app.controller('registrationController', [
    '$scope', function($scope) {
    // registration stuff
    }
  ]);
})(angular.module('bootstrapper'));
```

```javascript 
(function(app) {
  'use strict';

  app.controller('signinController', [
    '$scope', function($scope) {
    // sign in stuff
    }
  ]);
})(angular.module('bootstrapper'));
```