---
layout: post
title: Insert page specific CSS or JS using content_for
---

There might be cases where you want to insert some CSS or JS snippet into a specific page only , for example inserting Stripe JS and CSS only on the payment page, but not on other pages of the same controller.



If you insert the snippet into layout/application.html.erb , it will be available to all the pages, which might slow down the load time of other pages which doesn't use the script!

```html
<!-- app/views/layout/application.html.erb -->
<!DOCTYPE html>
<html>
  <head>
    <title>My awesome app</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
    <%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
      
     <!-- This Stripe script will be available to all pages even if they dont have payment form -->
     <script src="https://js.stripe.com/v3/"></script>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```



We can solve this by using a custom yield and [content_for](https://guides.rubyonrails.org/layouts_and_rendering.html#using-the-content-for-method) function. Before moving to the solution, let's look at how the "yield" function works.



Notice the `<%= yield %>` code in **application.html.erb** ? This is how our content in **index.html.erb** get inserted to the layout file : 

![yield](https://rubyyagi.s3.amazonaws.com/13-page-specific-css-js/yield.png)



## Solution

We can define a custom yield inside the `<head>` tag, and only fill in this custom yield on the payment page like this : 

```html
<!-- app/views/layout/application.html.erb -->
<!DOCTYPE html>
<html>
  <head>
    <title>My awesome app</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
    <%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    
    <!-- we can put a custom yield with parameter here -->
    <%= yield(:head) %>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```



Then in the payment page (payment.html.erb) : 

```html
<!-- payment.html.erb -->
<% content_for :head do %>
  <!-- header content specific to this payment page -->
  <script src="https://js.stripe.com/v3/"></script>
<% end %>

  
<h1>Checkout page</h1>
<span>Pls buy my product kthxbye</span>
```





Here's how the final layout is generated : 

![custom yield layout](https://rubyyagi.s3.amazonaws.com/13-page-specific-css-js/custom.png)



You can use multiple custom yield with parameters in header / footer of a page, and use the **content_for** block to show different content on different pages.



eg : `<%= yield :abcd %>` will correspond to `<% content_for :abcd do %>` .

<script async data-uid="d862c2871b" src="https://rubyyagi.ck.page/d862c2871b/index.js"></script>

## References

[Official Rails layout and rendering guides](https://guides.rubyonrails.org/layouts_and_rendering.html#)


