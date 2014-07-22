---
layout: post
title: "JQuery binding with the future in mind"
date: 2011-10-11 18:53
comments: true
categories: jQuery
keywords: jquery binding
---
This was something that confused me for a while when I started JQuery – binding.  You add a new html element and want to bind to a click event for example on that element. Sounds straightforward.

We have 2 divs, the first being static and the second being a placeholder that we will dynamically add more html elements to.
<!--more-->
```html
<div id='ClickMe'>Click on me!</div>
<div id='placeholder'></div>
```

We add our elements using JQuery like this

```javascript
$(function(){  
    var newItems = '<div id="ClickMeToo">Click on me as well!</div>';  
    $('#placeholder').html(newItems);  
}); 
```

When we browse to the page, as expected, we get the 2 divs.

Now we can add the click events:

```javascript
$(function(){  
  var newItems = '<div id="ClickMeToo">Click on me as well!</div>
   
  $('#placeholder').html(newItems);  
  $('#ClickMe').click(function () {  
    alert('I always work');  
  });  
 
  $('#ClickMeToo').click(function () {  
    alert('I don't work...');  
  });  
});
```

But the second click event does not work.  Why?  Well the answer is that the `ClickMeToo` element does not exist when the JQuery does it’s binding and the new div we added is left out in the cold.

To resolve this, all we need to do is to amend the code $(‘#ClickMeToo’).click(function () { to the following:

```javascript
$('#ClickMeToo').live('click', function () {  
  alert("I'm gonna work now!");  
}); 
```

The `live` method tells JQuery to bind for all existing and future matches to the selector.

EDIT: the `live` binding is now deprecated - use `on` instead!