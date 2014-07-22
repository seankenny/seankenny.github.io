---
layout: post
title: "AngularJS XSRF in .Net land - SlidingExpiration"
date: 2014-01-22 19:52
comments: true
categories: [.net, angularjs, web api, authentication]
---
So we have [XSRF built into our app](http://seankenny.me/blog/2014/01/20/angularjs-xsrf-in-net-land/) and all is well.  Now we set the slidingExpiration="true" in our web.config file:
```xml
<authentication mode="Forms">
  <forms loginUrl="~/Account/Login" timeout="2880" slidingExpiration="true" />
</authentication>
```
<!--more-->
This will automatically update the authentication cookie once the cookie is nearing it's expiry time.  That way, if a user is on the site continuously then they will not need to re-authenticate.  This is a security flaw but real world requirements sometimes mean we need to be flexible.

When ASP.Net detects the authentication cookie needs to be refreshed, it creates a new authentication cookie with a new value.  Now when we check the X-XSRF-TOKEN request header value sent to us by angular as part of a POST request, the values will not match up.  The user will start getting 401 exceptions after a time.

To get around this, we will need to implement the sliding exipiration functionality ourselves.

Remove the web.config `slidingExpiration="true"` attribute.

In the global.asax:

```c#
protected void Application_PostAuthenticateRequest(object sender, EventArgs e)
{
    var authCookie = HttpContext.Current.Request.Cookies[FormsAuthentication.FormsCookieName];
    SlidingExpirationCookie(authCookie);
}
private void SlidingExpirationCookie(HttpCookie authCookie)
{
    if (authCookie == null)
    {
        return;
    }

    var ticket = FormsAuthentication.Decrypt(authCookie.Value);
    var newTicket = FormsAuthentication.RenewTicketIfOld(ticket);
    if (newTicket == null || newTicket.Expiration == ticket.Expiration)
    {
        return;
    }

    var encryptedTicket = FormsAuthentication.Encrypt(newTicket);
    authCookie = new HttpCookie(FormsAuthentication.FormsCookieName, encryptedTicket)
    {
        Secure = FormsAuthentication.RequireSSL,
        Path = FormsAuthentication.FormsCookiePath,
        Domain = FormsAuthentication.CookieDomain,
        HttpOnly = true
    };

    if (ticket.IsPersistent)
    {
        authCookie.Expires = ticket.Expiration;
    }

    Response.Cookies.Add(authCookie);

    var csrfToken = new CsrfTokenHelper().GenerateCsrfTokenFromAuthToken(authCookie.Value);
    var csrfCookie = new HttpCookie("FORM-XSRF", csrfToken) // remember, we don't use the default XSRF-TOKEN cookie name
    {
        HttpOnly = false,
        Secure = authCookie.Secure,
        Path = authCookie.Path,
        Domain = authCookie.Domain
    };

    HttpContext.Current.Response.Cookies.Add(csrfCookie);
}
```

Now the authentication cooke and the XSRF cookie will stay in sync.