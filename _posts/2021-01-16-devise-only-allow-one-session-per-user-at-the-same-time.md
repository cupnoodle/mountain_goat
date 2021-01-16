---
layout: post
title: Devise only allow one session per user at the same time
date: 2021-01-16 23:42 +0800
---


Sometimes for security purpose, or you just dont want users to share their accounts, it is useful to implement a check to ensure that only one login (session) is allowed per user at the same time.



To check if there is only one login, we will need a column to store information of the current login information. We can use a string column to store a token, and this token will be randomly generated each time the user is logged in, and then we compare if the current session's login token is equal to this token.



Assuming your users info are stored in the **users** table (you can change to other name for the command below), create a "current_login_token" column for the table :

```
rails g migration AddCurrentLoginTokenToUsers current_login_token:string
```



This would generate migration file below  :

```ruby
class AddCurrentLoginTokenToUsers < ActiveRecord::Migration[6.1]
  def change
    add_column :users, :current_login_token, :string
  end
end
```



Now run **rails db:migrate** to run this migration.

```
rails db:migrate
```



Next, we would need to override the **create** method of SessionsController from Devise, so that we can generate a random string for the **current_login_token** column each time the user signs in.



If you don't already have a custom SessionsController created already, you can generate one using this command : 

```
rails g devise:controllers users -c=sessions
```

(Assuming the model name is **user**)



The -c flag means the controller to generate, as we only need to generate SessionsController here. (You can read more about the [devise generator command here](https://github.com/heartcombo/devise/wiki/Tool:-Generate-and-customize-controllers))



This will generate a controller in **app/controllers/users/sessions_controller.rb** , and this will override the default Devise's SessionsController.



Next, we are going to override the "**create**" method (the method that will be called when user sign in successfully) inside **app/controllers/users/sessions_controller.rb** : 

```ruby
class Users::SessionsController < Devise::SessionsController
  skip_before_action :check_concurrent_session

  # POST /resource/sign_in
  def create
    super
    # after the user signs in successfully, set the current login token
    set_login_token
  end

  private
  def set_login_token
    token = Devise.friendly_token
    session[:login_token] = token
    current_user.current_login_token = token
    current_user.save
  end
end

```



We use [Devise.friendly_token](https://github.com/heartcombo/devise/blob/45b831c4ea5a35914037bd27fe88b76d7b3683a4/lib/devise.rb#L492) to generate a random string for the token, then store it into the current session (**session[:login_token]**) and also save to the current user's **current_login_token** column.



The **skip_before_action :check_concurrent_session** will be explained later, put it there for now.



Next, we will edit the **application_controller.rb** file (or whichever controller where most of the controllers that require authorization are inherited from).

```ruby
class ApplicationController < ActionController::Base
  # perform the check before each controller action are executed
  before_action :check_concurrent_session

  private

  def check_concurrent_session
    if already_logged_in?
      # sign out the previously logged in user, only left the newly login user
      sign_out_and_redirect(current_user)
    end
  end

  def already_logged_in?
    # if current user is logged in, but the user login token in session 
    # doesn't match the login token in database,
    # which means that there is a new login for this user
    current_user && !(session[:login_token] == current_user.current_login_token)
  end
end
```



As we have wrote **before_action :check_concurrent_session** in the application_controller.rb , all controllers inherited from this will run the **check_concurrent_session** method before the controller action method are executed, this will check if there is any new login session initiated for the same user.



We want to exempt this check when user is on the login part, ie. sessions#create , hence we put a **skip_before_action :check_concurrent_session** to skip this check for the Users::SessionController class earlier.



Here's a demo video of this code : 

![demo](https://rubyyagi.s3.amazonaws.com/18-devise-limit-one-user/single-login-demo.gif)



Notice that after the second login attempt, the previously logged in user gets logged out after clicking a link (executing a controller action).

<script async data-uid="4776ba93ea" src="https://rubyyagi.ck.page/4776ba93ea/index.js"></script>


