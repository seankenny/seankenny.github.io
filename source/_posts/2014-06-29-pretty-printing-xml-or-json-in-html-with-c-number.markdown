---
layout: post
title: "Pretty printing Xml or Json in HTML with c#"
date: 2014-06-28 12:10
comments: true
categories: [C#]
keywords: pretty print, xml format, json format, c#, razor
description: How to pretty print or format xml and json in html using c#
---
There are a few different ways to pretty print objects as html in C#.  We have a requirement to pretty print what might be XML or Json.  The following extension method does the trick:
 <!--more-->
```c#
public static string PrettyPrint(this string serialisedInput)
{
  if (string.IsNullOrEmpty(input))
  {
    return input;
  }
 
  try
  {
    return XDocument.Parse(input).ToString();
  }
  catch (Exception) { }
 
  try
  {
    var t = JsonConvert.DeserializeObject<object>(input);
    return JsonConvert.SerializeObject(t, Formatting.Indented);
  }
  catch (Exception) { }
 
  return input;
}
```

In our case, we will have previously serialised the object into a datastore:

```c#
var serialisedItem = "{\"name\":\"sean\",\"roles\":[{\"roleName\":\"editor\"},{\"roleName\":\"admin\"}]}";
```

or for those of us unfortunate to work in 1998:

```c#
var serialisedItem = "<?xml version=\"1.0\" encoding=\"UTF-8\" ?><Name>sean</Name><Roles><RoleName>editor</RoleName></Roles><Roles><RoleName>admin</RoleName></Roles>";
```

The Html/Razor syntax:

```html
<pre>@serialisedItem.PrettyPrint()</pre>
```

Leads to (ta da!):
```html
{
    "name" : "sean",
    "roles" : [{
          "roleName" : "editor"
        }, {
          "roleName" : "admin"
        }
    ]
}
```

or to:

```html
<?xml version="1.0" encoding="UTF-8" ?>
<Name>sean</Name>
<Roles>
    <RoleName>editor</RoleName>
</Roles>
<Roles>
    <RoleName>admin</RoleName>
</Roles>
```