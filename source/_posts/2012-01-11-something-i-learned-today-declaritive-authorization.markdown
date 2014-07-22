---
layout: post
title: "Something I learned today – Declaritive Authorization"
date: 2012-01-11 18:53
comments: true
categories: rails
---
Well not exactly true.  We’ve been using this great gem for a while now on our first venture into Ruby on Rails.

Ryan Bates over at RailsCasts has a great session on the topic.  We implemented the basic vanilla version on our RESTful controller actions and models and everything seemed to run fine after making a few tweaks.
<!--more-->
Then a funny thing happened.  One of the guys ran the db:migrate on a fresh database and when he tried to access a page that had an AJAX call on it, the ajax failed.  Looking at the console, we got an ActiveRecord exception.

```ruby
Resource Load (0.2ms)  SELECT `resources`.* FROM `resources` WHERE (`resources`.`id` = 1) LIMIT 1
ActiveRecord::RecordNotFound (Couldn't find Resource without an ID):
```

The funny thing was that the action being accessed did not call anything relating to the Resource model.  It’s an Ajax/JSON action just for a dropdown filter on the page.

It took a while to figure and it boiled down to RTFM!  Declaritive Authorizaton will always load up the model for each action in your locked down controller in order to figure out if the thread has access.  News to me although it does mean that we were able to remove all the model finds, etc., in our RESTful actions show, new, edit..).

To fix this particular issue, all that is required is to tell create a dummy model prior to DA kicking in.

```ruby
class Admin::ResourcesController &lt; Admin::BaseController
  before_filter :new_resource, :only => :for_resource_type_id
  filter_resource_access
   
  def for_resource_type_id
    @resourcegroups = ResourceGroup.where('resource_type_id = ?', params[:id])
    render :json => @resourcegroups
  end
   
  private
  def new_resource
    @resource = Resource.new
  end
end
```

Easy when you know how.