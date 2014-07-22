---
layout: post
title: "Preloading AngularJS data in .Net land"
date: 2013-09-29 17:17
comments: true
categories: [.net, angularjs]
---
When an angularjs site initially starts up, it can take a little time for all the various HTTP requests to complete.

HTTP requests

 * GET Index.cshtml
 * GET the various JS and CSS files (hopefully only one minified version of each)
 <!--more-->
Now angularjs starts up.  Usually the first thing an angular controller will do it to get data for the UI:
```js
app.controller('MyController', ['$scope', '$http', function($scope, $http){
  $http.get('api/users/1', function(response){
    $scope.user = response.user;
  ));
  
  // other stuff
}]);
```

This is yet another HTTP request.  However it is one that we do not have to make if we make use of `ng-init`.

Create the ViewModel/DataModel:
```c#
public class UserDataModel
{
  public string FirstName { get; set; }
  public string LastName { get; set; }
}
```

In the MVC controller:
```c#
public ViewResult Index()
{
  var dataModel = new UserDataModel
  {
    FirstName = "Joe",
    LastName = "Shmoe"
  };
  
  return View(dataModel);
}
```

In the html:
```html
@model UserDataModel
<div ng-controller='MyController' ng-init="user={name:'@Model.FirstName',lastName:'@Model.LastName'}">
```

Our angularjs controller changes to:
```js
app.controller('MyController', ['$scope', '$http', function($scope, $http){
  // other stuff
}]);
```

And now we have the `user` object in our controller `$scope`.

Just be aware that the ng-init is on the same HTML element as the ng-controller so it will run after the controller is instantiated.  The object will get put on the controller `$scope` but will be available immediately in the controller init path.  You can use the [`$scope.evalAsync` function](http://docs.angularjs.org/api/ng.$rootScope.Scope#$evalAsync) in that case which will re-evaluate the $scope.user on the $digest cycle.