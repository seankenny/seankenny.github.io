---
layout: post
title: "Using git, npm and bower behind a firewall"
date: 2014-08-05 17:50:53 +0100
comments: true
categories: [git, bower, npm]
---
> Self-post for me as I have already had to re-learn this 3 times now...

Company firewalls and policies can prevent npm and bower installing correctly.

###Switching protocols
Some companies will blacklist the git protocol => `git://github.com/modulename` so we can change this to use http using:

```bash
git config --global url."http://".insteadOf git://
```

###Setting git to use a proxy
If your company runs a proxy, set git to use it.  Get your proxy from your IE settings.  Set your git config to use the proxy here:

```bash
export HTTP_PROXY=http://mycompanyproxy:port
export HTTPS_PROXY=http://mycompanyproxy:port

git config --global http.proxy http://mycompanyproxy:port
git config --global https.proxy http://mycompanyproxy:port
```

Bower will need futher updates as it uses lower case settings on the env variables:
```bash
export http_proxy=http://mycompanyproxy:port
export https_proxy=http://mycompanyproxy:port
```