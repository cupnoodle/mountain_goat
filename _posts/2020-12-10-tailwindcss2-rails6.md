---
layout: post
title: How to install TailwindCSS 2 on Rails 6
---

[TailwindCSS v2.0](https://blog.tailwindcss.com/tailwindcss-v2) was released on 19 November 2020 🎉, but it had a few changes that requires some tweak to install and use it on Rails 6, such as the requirement of PostCSS 8 and Webpack 5. As of current (10 December 2020), Rails 6 comes with webpack 4 and PostCSS 7, which doesn't match the requirement of TailwindCSS v2.



If we install TailwindCSS v2 directly on a Rails 6 app using `yarn add tailwindcss `, we might get a error (PostCSS plugin tailwindcss requires PostCSS 8) like this when running the server : 

![postcss8 error](https://rubyyagi.s3.amazonaws.com/17-tailwindcss2-rails6/tailwind-postcss-error.png)


<br>


This tutorial will guide you on **how to install and use TailwindCSS 2 on Rails 6 using new Webpacker version(v5+)**. If webpack / webpacker v5+ breaks your current Rails app javascript stuff, I recommend using the compability build of Tailwind [following this tutorial](https://rubyyagi.com/tailwind-css-on-rails-6-intro/#installing-tailwind-css-on-your-rails-app).



## Update webpacker gem and npm package

Currently the latest [Webpacker NPM package](https://www.npmjs.com/package/@rails/webpacker) is not up to date to the [master branch of Webpacker repository](https://github.com/rails/webpacker), we need to use the Github master version until Rails team decide to push the new version of webpacker into NPM registry.



![webpacker release](https://rubyyagi.s3.amazonaws.com/17-tailwindcss2-rails6/webpacker_release.png)



To do this, we need to uninstall the current webpacker in our Rails app : 

```bash
yarn remove @rails/webpacker
```



And install the latest webpacker directly from Github (**notice without the "@"**) :

```bash
yarn add rails/webpacker
```



Here's the difference in package.json :

![package.json](https://rubyyagi.s3.amazonaws.com/17-tailwindcss2-rails6/package_compare.png)



Next, we need to do the same for the Webpacker gem as well, as they haven't push the latest webpacker code in Github to RubyGems registry.



In your **Gemfile**, change the webpacker gem from :

```ruby
gem 'webpacker', '~> 4.0'
```



**to:** 

```ruby
gem "webpacker", github: "rails/webpacker"
```



then run `bundle install` to install the gem from its Github repository. You should see the webpacker gem is version 5+ now : 

![webpacker master](https://rubyyagi.s3.amazonaws.com/17-tailwindcss2-rails6/webpacker-master.png)





## Install Tailwind CSS 2

After updating the webpacker gem, we can install TailwindCSS (and its peer dependency) using yarn :

```bash
yarn add tailwindcss postcss autoprefixer
```



We can use webpacker for handling Tailwind CSS in our Rails app, create a folder named **stylesheets** inside **app/javascript** (the full path would be **app/javascript/stylesheets**).



Inside the app/javascript/stylesheets folder, create a file and name it “**application.scss**”, then import Tailwind related CSS inside this file :

```css
/* application.scss */
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";
```



The file structure looks like this :

![application scss file structure](https://rubyyagi.s3.amazonaws.com/6-tailwind-intro/tailwind_import.png)



Then in **app/javascript/packs/application.js** , require the application.scss :

```javascript
// app/javascript/packs/application.js

require("@rails/ujs").start()
require("turbolinks").start()

require("stylesheets/application.scss")
```



Next, we have to add **stylesheet_pack_tag** that reference the app/javascript/stylesheets/application.scss file in the layout file (app/views/layouts/application.html.erb). So the layout can import the stylesheet there.

```html
<!-- app/views/layouts/application.html.erb-->

<!DOCTYPE html>
<html>
  <head>
    <title>Bootstraper</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    
    <!-- This refers to app/javascript/stylesheets/application.scss-->
    <%= stylesheet_pack_tag 'application', 'data-turbolinks-track': 'reload' %>

    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>
....
```





## Add TailwindCSS to PostCSS config

Next, we will add TailwindCSS into the **postcss.config.js** file (located in your Rails app root) :

```javascript
// postcss.config.js

module.exports = {
  plugins: [
    require("tailwindcss"),
    require('postcss-import'),
    require('postcss-flexbugs-fixes'),
    require('postcss-preset-env')({
      autoprefixer: {
        flexbox: 'no-2009'
      },
      stage: 3
    })
  ]
}
```



If you are not using the default file name for TailwindCSS configuration file (tailwind.config.js) , you need to specify it after the tailwindcss require : 

```javascript
// postcss.config.js

module.exports = {
  plugins: [
    // if you are using non default config filename
    require("tailwindcss")("tailwind.config.js"),
    require('postcss-import'),
    require('postcss-flexbugs-fixes'),
    require('postcss-preset-env')({
      autoprefixer: {
        flexbox: 'no-2009'
      },
      stage: 3
    })
  ]
}
```



Now we can use the Tailwind CSS classes on our HTML elements!

```html
<h1>This is static#index</h1>
<div class="bg-red-500 w-3/4 mx-auto p-4">
  <p class="text-white text-2xl">Test test test</p>
</div>
```



![demo](https://rubyyagi.s3.amazonaws.com/6-tailwind-intro/tailwind_preview.png)

You can refer to [Tailwind CSS documentation](https://tailwindcss.com/docs) for the list of classes you can use.



For further Tailwind CSS customization and configuring PurgeCSS, you can [refer this post](https://rubyyagi.com/tailwind-css-on-rails-6-intro/#changing-tailwind-css-default-config).



## References

Thanks Kieran for writing the [Tailwind CSS 2 upgrade guide](https://www.kieranklaassen.com/upgrade-rails-to-tailwind-css-2/).

<script async data-uid="d862c2871b" src="https://rubyyagi.ck.page/d862c2871b/index.js"></script>


