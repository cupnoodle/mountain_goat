---
layout: post
title: 'Rails tip: use #presence to get params value, or default to a preset value'
date: 2021-08-21 16:55 +0800
---

Here's a method I encountered recently at my work and I found it might be useful to you as well.


There might be some attribute which you want to set to a default parameter if the user didn't supply any, eg:

```ruby
# if the parameters is nil or blank, we will default to french fries
sides = params[:sides].present? ? params[:sides] : "french fries"
```



This one liner looks concise, but we can make it even shorter like this :

```ruby
sides = params[:sides].presence || "french fries"
```



The **presence** method will return nil if the string is a blank string / empty array or nil , then the `||` (double pipe) will use the value on the right side ("french fries").

```ruby
''.presence
# => nil
```



If the string is not blank, **.presence** will return the string itself.



When **params[:sides]** == ""  :

```ruby
sides = params[:sides].presence || "french fries"
# => "french fries"
```



When **params[:sides]** == "corn" , 

```ruby
sides = params[:sides].presence || "french fries"
# => "corn"
```



Documentation : [http://api.rubyonrails.org/classes/Object.html#method-i-presence](http://api.rubyonrails.org/classes/Object.html#method-i-presence)

<script async data-uid="4776ba93ea" src="https://rubyyagi.ck.page/4776ba93ea/index.js"></script>


