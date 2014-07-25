---
layout: post
title: "Custom OWIN IClaimsPrincipal with Ninject"
date: 2014-07-25 17:33:19 +0100
comments: true
categories: [c#, webapi, OWIN]
---
Having recently started using OWIN to handle authorization, I've had to change a few things throughout my web api projects. To have the user's identity available, I usually create something like:
<!--more-->
```c#
    public class MyPrincipal : IMyPrincipal
    {
        private readonly ClaimsPrincipal claimsPrincipal;

        public MnsPrincipal(ClaimsPrincipal claimsPrincipal)
        {
            this.claimsPrincipal = claimsPrincipal;
        }

        public string Id
        {
            get
            {
                var claim = claimsPrincipal.Claims.FirstOrDefault(c => c.Type == ClaimTypes.NameIdentifier);
                return claim != null ? claim.Value : string.Empty;
            }
        }
    }
    
    public interface IMyPrincipal
    {
        string Id { get; }
    }
```

This will add the `ClaimsPrinciple` using composition allowing me to use Ninject like this:

```c#
    kernel.Bind<IMyPrincipal>()
        .ToMethod(ctx => new MyPrincipal(HttpContext.Current.GetOwinContext().Authentication.User))
        .InRequestScope();

```

Now I can get to the logged in user's Id and claims/roles by using constructor injection:

```c#
    public class UsersController : ApiController
    {
        private readonly IMyPrincipal myPrinciple;

        public UsersController(IMyPrincipal myPrinciple)
        {
            this.myPrinciple = myPrinciple;
        }
        
        public HttpResponseMessage Get()
        {
            return Request.CreateResponse(HttpStatusCode.OK, myPrinciple.Id); // will contain my NameIdentifier claim type
        }
    }
```

It also allows me to get to the principal when I am no longer in the web api project. My repository/service/flavour of the month class simply has a contructor with the `IMyPrincipal` argument.

Previously, using FormsAuth, the principal was available as an `IPrincipal` using `HttpContext.Current.User` meaning we could use inheritance.  The OWIN approach leads us down the (currently) preferred composition route :-)