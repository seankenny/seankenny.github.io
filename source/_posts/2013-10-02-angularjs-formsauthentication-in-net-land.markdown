---
layout: post
title: "AngularJS FormsAuthentication in .Net land"
date: 2013-10-02 18:31
comments: true
categories: [.net, angularjs, web api, authentication]
---
There are a number of ways to authenticate an angularJS application.  This is the one I am currently using.  It uses a `FormsAuthentication` cookie in the same way as a regular ASP.Net WebForms/MVC app would.

The structure is basically Web Api, a single MVC home controller with an Index.cshtml and angular.
<!--more-->
The HTML sign in form:
```xml
<form name="form" novalidate>
    <input type="text" ng-model="user.userName" />
    <input type="password" ng-model="user.password" />
    <input type="submit" value="Sign In" data-ng-click="signin(user)">
</form>
```

Clicking the 'Sign In' calls the Angular Sign In function:

```js
$scope.signin = function(user) {
    $http.post('api/accounts/signin', user)
        .success(function(data, status, headers, config) {
            user.authenticated = true;
            $rootScope.user = user;
            $location.path('/');
        })
        .error(function(data, status, headers, config) {
            user.authenticated = false;
            $rootScope.user = {};
        });
};
```

Which calls the Web Api Accounts Controller
```c#
public class AccountsController : ApiController
{
    [HttpPost]
    public HttpResponseMessage SignIn(UserDataModel user)
    {
        if (this.ModelState.IsValid)
        {
            // authenticate the user however you wish to do so here....
            if (authenticated)
            {
                var response = this.Request.CreateResponse(HttpStatusCode.Created, true);
                FormsAuthentication.SetAuthCookie(user.UserName, false);
                return response;
            }
            return this.Request.CreateErrorResponse(HttpStatusCode.Forbidden);
        }
        return this.Request.CreateErrorResponse(HttpStatusCode.BadRequest, this.ModelState);
    }
}

// USER DATAMODEL:
public class UserDataModel
{
    public string UserName { get; set; }
    public string Password { get; set; }
}
```

Which adds an encrypted cookie to the response.

So now we have a `$rootScope.user` object that has an `authenticated` property set to true/false.  This allows us to hide/show menus, etc.  Obviously this can be tampered with so we secure things by locking down any server side code we want to:

```c#
[Authorize]
public class AccountsController : ApiController
{
    [HttpPost]
    public HttpResponseMessage SomethingPrivate(UserDataModel user)
    {
        // this will only be reached if the FromsAuthentication cookie arrives in the request.  The [Authorize] filter will redirect to a HTTP 403 Forbidden Request if none is present.
    }
}
```

The sign out is:
```js
$scope.signin = function(user) {
    $http.post('api/accounts/signout').
        success(function(data, status, headers, config) {
            localStorageService.clearAll();
            $rootScope.user = {};
            $location.path('/');
        });
};
```

Which calls the Web Api Accounts SignOut Controller Action
```c#
public class AccountsController : ApiController
{
    [HttpPost]
    public HttpResponseMessage SignOut(UserDataModel user)
    {
        if (this.ModelState.IsValid)
        {
            var response = this.Request.CreateResponse(HttpStatusCode.Created, true);
            FormsAuthentication.SignOut();
            
            return response;
        }
        return this.Request.CreateErrorResponse(HttpStatusCode.BadRequest, this.ModelState);
    }
}
```

This will remove the FormsAuthentication cookie from the user's browser.  The angularjs `$rootScope.user` will be reset and we are now logged out.

The only other thing to be aware of is when the user refreshed the page, etc.  We need to authomatically 'sign in' the user if the cookie is present.  This is done on our MVC Controller that supplies the Index.cshtml page.

```c#
public class HomeController : Controller
{
    [HttpPost]
    public ViewResult Index()
    {
        var dataModel = new BootStrapperDataModel
        {
            Authenticated = Request.IsAuthenticated ? "true": "false",
        };
        
        if (Request.IsAuthenticated)
        {
            dataModel.UserName = Thread.CurrentPrincipal.Identity.Name;
        }
        
        return View(dataModel);
    }
}
```

And the update to the Index.cshtml to update the `$rootScope.user`:

```html
@model BootStrapperDataModel

<div ng-init="user={authenticated:'@Model.Authenticated',userName:'@Model.UserName'}">
    <div ng-controller="MyController">
```

Check out the [post](http://seankenny.me/blog/2013/10/02/preloading-angularjs-data-in-net-land/) on ng-init to see how to hook this up with angularjs in more detail.