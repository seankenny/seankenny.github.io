---
layout: post
title: "Infinite scrolling with JQuery"
date: 2012-05-11 18:52
comments: true
categories: jQuery
---
Recently I was asked to include an infinite scroll for a client’s twitter account on their homepage.  I was surprised at how easy it turns out to be.

In a nutshell, it consists of:

Use AJAX to hit Twitter’s timeline API to get the tweets
<!--more-->
Use JQuery to detect when the user is scrolling

Use a little bit of maths to figure out when to trigger another page of tweets

First, we add a link to grab JQuery from the Google CDN.

Next, add in placeholders for the tweets

```html
<body>
  <ul id='tweets'></ul>
</body>
```

Now the call to get the tweets from Twitter and add them as items to our list (I’m using Stephen Fry as the test subject as he tweets LOADS):

```javascript
$.ajax({
  dataType: "json",
  url: 'http://api.twitter.com/1/statuses/user_timeline.json',
  data: {
    id: 'stephenfry'
    skip_user: true,
    page: pageNumber
  },
  success: function(response){
    for(var i=0;i&lt;response.length; i++){
      $('#tweets').append('<li>' + response[i].text + '</li>');
    }
});
```

Note that we are passing in the page number.  This is important for the next few bits.

Now we add the JQuery to load the page on opening.

```javascript
$(function(){
  getPagedTweets(1);
});
```

For the infinite scroll, we need to do a few things.  First, we need to be able to pass the next page to Twitter’s API so we need a mechanism to store it.  We can use a HTML5 data item to do this:

```html
<ul id="tweets" data-nextpage="2"></ul>
```

Next, we need a nextPage function to get and set this value as well as call the getPagedTweets function we already have:

```javascript
function nextPage(){
  var pageNumber = parseInt($('#tweets').attr('data-nextpage'));
  getPagedTweets(pageNumber);
  $('#tweets').attr('data-nextpage', pageNumber + 1);
};
```

Finally, we need the JQuery to trigger an event when the user scrolls to the bottom of the page:

```javascript
$(function(){
  getPagedTweets(1);
  $(window).scroll(function(){
    if($(window).scrollTop() &gt;= $(document).height() - $(window).height()){
      nextPage();
    };
  });
});
```

And that should be that.

Note the page should be resized to ensure that scrollbars are visible otherwise this will not work as the user needs to scroll in order for the next page call to work.