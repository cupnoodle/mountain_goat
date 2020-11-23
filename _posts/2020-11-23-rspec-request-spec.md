---
layout: post
title: Introduction to API testing using RSpec Request Spec
---

If you are new to Rails testing and RSpec, I recommend going through the [beginner tutorial for Rails testing using RSpec and Capybara](https://rubyyagi.com/intro-rspec-capybara-testing/) before proceeding with this tutorial. RSpec is the testing framework we will be using for this tutorial.



Sometimes your Rails app might have API endpoints exposed for mobile apps, separate web frontend like React or even for third party to consume, it is important to write test for these API endpoints to ensure they are returning the correct response and also perform actions as specified.



For this tutorial, we will create a Rails app with API mode (no UI) and write specs to test the API endpoints, you can use your own Rails app and follow along as long as it has API endpoints exposed to public.



This article will cover testing fixed JSON response, API that handles CRUD operation of a model, and authentication. To follow along this tutorial, I recommend downloading a GUI HTTP API tool like [Postman](https://www.postman.com/downloads/), [Insomnia](https://insomnia.rest) or [Paw](https://paw.cloud) to create and send HTTP request to your own Rails API.

**Table of contents**
1. [Creating the sample Rails app](#creating-the-sample-rails-app)
2. [Install RSpec-rails](#install-rspec-rails)
3. [Create a health check controller to return fixed JSON response](#create-a-health-check-controller-to-return-fixed-json-response)
4. [Write your first request spec](#write-your-first-request-spec)
5. [Ensure the test actually work by making it fail](#ensure-the-test-actually-work-by-making-it-fail)
6. [Create a CRUD scaffold](#create-a-crud-scaffold)
7. [Write test for CRUD actions](#write-test-for-crud-actions)
8. [Testing authentication](#testing-authentication)
9. [Reference](#reference)


## Creating the sample Rails app

First, let's create a API mode Rails app using **rails new**

`rails new bookmarkapi --api -T`



`--api` flag is used to tell Rails we will use it mainly for API purpose, this will configure the generators to skip creating views and helpers when we create a new controller or scaffold, and the  `ApplicationController` class will inherit from the more lightweight `ActionController::API` class instead of the usual `ActionController::Base `class.



[From Rails official documentation of ActionController::API](https://api.rubyonrails.org/classes/ActionController/API.html)  :

> it doesn't include a number of features that are usually required by browser access only: layouts and templates rendering, flash, assets, and so on. This makes the entire controller stack thinner, suitable for [`API`](https://api.rubyonrails.org/classes/ActionController/API.html) applications. 



You can use any app name you want,  the `-T` flag is used to tell Rails not to use MiniTest. By default, Rails will use a test framework called 'MiniTest' if we didn't specify `-T` flag, we want to use **RSpec** as the test framework instead of MiniTest.



Next, create the database of the Rails app so that we can run the app later :

```bash
cd bookmarkapi
rake db:create
```



## Install rspec-rails

Now, we will install the rspec-rails into our Rails app.



**rspec-rails** is the rspec testing framework, plus some adopted Rails magic. For example when you run `rails generate model User` to create user , it will also automatically create a model test file for that user : **user_spec.rb**  .



Include it in the **Gemfile**, inside the **:development, :test**  group like this :

```ruby
# Gemfile

# ...
group :development, :test do
  gem 'rspec-rails'
end
```



Then run `bundle install` to install it.



After installing these gems, we still need to install RSpec into our app using 

`rails generate rspec:install`  .



This will create a few boilerplate files, and update some configuration in our Rails app.



## Create a health check controller to return fixed JSON response

Usually for API endpoints, I will create a health check endpoint that returns a fixed HTTP 200 (OK) response so the requester can know that the API is working as intended.



In this section we will create an endpoint that return a fixed JSON response, and write a test to verify the content of the JSON response and also HTTP status.



Let's create a controller **HealthController** with just a single action **index** : 

`rails generate controller health index`



This will create **health_controller.rb**  and the associated route to the index action (eg: GET '/health/index').



Open **health_controller.rb**, and edit the index action to return a JSON response with HTTP 200 status.

```ruby
# health_controller.rb

class HealthController < ApplicationController
  def index
    render json: { status: 'online' }, status: 200
  end
end

```



## Write your first request spec

If you have created the Rails app in API mode, RSpec will auto generate request spec files for you when you create a new controller using `rails generate controller` .



This should be located in **spec/requests/health_request_spec.rb**.

![pre-generated request spec](https://rubyyagi.s3.amazonaws.com/14-rspec-request-spec/default_request.png)



If this is not auto generated on your side, then you would need to create a **health_request_spec.rb** file, and place it under **spec/requests** folder.



Modify the content of the **health_request_spec.rb** file to look like this :

```ruby
require 'rails_helper'

RSpec.describe "Healths", type: :request do
  describe "GET /index" do
    it "returns http success" do
      # this will perform a GET request to the /health/index route
      get "/health/index"
      
      # 'response' is a special object which contain HTTP response received after a request is sent
      # response.body is the body of the HTTP response, which here contain a JSON string
      expect(response.body).to eq('{"status":"online"}')
      
      # we can also check the http status of the response
      expect(response.status).to eq(200)
    end
  end
end
```



Then in your terminal, navigate to your Rails app root path, then run 

`rspec spec/requests/health_request_spec.rb`



You should see the test pass successfully :

![first_successful_request_spec](https://rubyyagi.s3.amazonaws.com/14-rspec-request-spec/first_successful_spec.png)

Congratulation! You have written your first RSpec request spec (if you haven’t already before 😬) . The convention of RSpec is that all test files must end with “_spec” in their filename.



Below is the explanation of what each line does in the test file.

```
require 'rails_helper'
```

This line will load the necessary test configuration from **spec/rails_helper.rb**, which is needed for the test to run.



```
RSpec.describe "Healths", type: :request do
```

This line is how we start writing the test, the test code usually reside inside the describe block.



`describe "GET /index" do`

We used another describe block inside the main describe block, to further categorize. This is useful when you have multiple routes in the same test file like GET /bookmark/2 and PUT /bookmark/2 .



You can replace ‘Healths’ with any string you want, it is for your own documentation purpose.


The **type: :request** will tell RSpec that this is a system test.



```ruby
it "returns http success" do
```

This line is also for documentation purpose, you replace the string with any string you want, but note that the **expect()** assertion will only works inside a **it** or **scenario** block.



```ruby
get "/health/index"
```

This will perform a GET request to the /health/index route. Similarly, you can also use "post", "put", "patch", "delete" methods to send POST, PUT, PATCH and DELETE HTTP request. These methods are from Rails's [ActionDispatch::Integration::RequestHelpers](https://api.rubyonrails.org/classes/ActionDispatch/Integration/RequestHelpers.html) module.



**response**  is a variable representing the HTTP response we receive after making a HTTP request, its class is [ActionDispatch::TestResponse](https://api.rubyonrails.org/classes/ActionDispatch/TestResponse.html), we can use its' **body** method to get the response body to check its content,  **status** attribute to get the HTTP status code, and **headers** attribute to get the HTTP headers.



```ruby
expect(response.body).to eq('{"status":"online"}')

expect(response.status).to eq(200)
```

 This will tell RSpec to ensure the response body is equal to the JSON string **{"status": "online"}**, and the response status is 200 (HTTP 200 means ok).



The above syntax might look magical at first, it is equivalent to

```ruby
expect(response.body).to(eq('{"status":"online"}'))

expect(response.status).to(eq(200))
```

With ruby magic, we can omit the parentheses for the **to** method.



Sometimes when testing a deeply nested JSON string response, we can make it easier to read by parsing it to a Ruby hash using **JSON.parse** like this :

```ruby
# by default, JSON has string key like {"status": "online"},
# we then convert the key to symbol, {status: 'online'} for better readability, 
# and type less two quotes character

json = JSON.parse(response.body).deep_symbolize_keys

# easier to read in ruby hash form, compared to a full JSON string
expect(json).to eq({
  order_id: 123,
  user: {
    id: 456,
    name: 'Asriel Dreemurr'
  },
  product: {
    id: 789,
    name: 'Butterscotch Pie'
  }
})

# this is a lot harder to read
expect(response.body).to eq('{"order_id":123,"user":{"id":456,"name":"Asriel Dreemurr"},"product":{"id":789,"name":"Butterscotch Pie"}}')
```



## Ensure the test actually work by making it fail

How do we know our test actually work? What if the test we wrote always passes because we didn’t check for enough condition? This might render the test useless as it doesn’t guard us from breaking the API.



To ensure the test actually work, we can try break it on purpose. As our test check for the JSON status online, we can modify the controller to return a different JSON, for example:  {status: 'up'}.



Then run the same test again :

```
rspec spec/requests/health_request_spec.rb
```



Sure enough, the test fails :

![failed_test](https://rubyyagi.s3.amazonaws.com/14-rspec-request-spec/failed_test.png)



If you are wondering where does the “Healths GET /index returns http success” string comes from, it is from the strings you used in the **describe** and **it** blocks :



![failure documentation](https://rubyyagi.s3.amazonaws.com/14-rspec-request-spec/failure_document.png)





The strings you used in the **describe** and **it** / **scenario** block will serve as documentation when your test goes wrong, to help you pinpoint which part of your test has failed.



## Create a CRUD scaffold

Next, we are going to create scaffold (model + controller) so that we can write test for basic CRUD features (create, read, update, delete). Usually scaffold will work out of the box without needing to test it, nevertheless in this section I will try explain my experience / thought on how to test CRUD.



Let’s create a sample scaffold, I used “Bookmark”, with two string properties : “title” and “url”.

```
rails generate scaffold Bookmark title:string url:string
```



Running this command generate the model, controller and prefilled request specs file for the bookmark routes.



As we don’t want user to input empty values into title and url column, open up **db/migrate/…._create_bookmarks.rb** and add ‘**null: false**’ to them



```ruby
# 20201120102131_create_bookmarks.rb
class CreateBookmarks < ActiveRecord::Migration[6.0]
  def change
    create_table :bookmarks do |t|
      t.string :title, null: false
      t.string :url, null: false

      t.timestamps
    end
  end
end

```



then run database migration :

```
rake db:migrate
```



Next, open the bookmark model file in **app/models/bookmark.rb** , and add validation for presence (not null), and disallow blank value :

```ruby
# app/models/bookmark.rb
class Bookmark < ApplicationRecord
  validates :title, presence: true, allow_blank: false
  validates :url, presence: true, allow_blank: false
end
```



The scaffold generates a **bookmarks_controller.rb** file which comes with a few action we can test with : 

```ruby
# app/controllers/bookmarks_controller.rb
# GET /bookmarks
# ....
  def index
    @bookmarks = Bookmark.all

    render json: @bookmarks
  end

  # GET /bookmarks/1
  def show
    render json: @bookmark
  end

  # POST /bookmarks
  def create
    @bookmark = Bookmark.new(bookmark_params)

    if @bookmark.save
      render json: @bookmark, status: :created, location: @bookmark
    else
      render json: @bookmark.errors, status: :unprocessable_entity
    end
  end
#...
```



Run `rails server` , then use a HTTP API tool like [Postman](https://www.postman.com/downloads/), [Insomnia](https://insomnia.rest) or [Paw](https://paw.cloud) to generate a HTTP request to the rails server running at localhost:3000 .



Here I send a JSON containing url and title of a bookmark object I want to create, and send it over via POST request to the **/bookmarks/** endpoint, which calls the **create** action.

![create request manually](https://rubyyagi.s3.amazonaws.com/14-rspec-request-spec/create_bookmark_manually.png)



As the url and title of the bookmark object is valid, the API returns back the JSON of the created bookmark with status 201 (:created).



Next, let's create a bookmark again , but with a blank URL :

![create bookmark fail with error](https://rubyyagi.s3.amazonaws.com/14-rspec-request-spec/invalid_create_bookmark.png)



As the url is blank and thus invalid, the API returns back a JSON containing error message for the url key, and a 422 Unprocessable Entity HTTP status.



In the next section, we will write request spec to simulate these two situations (valid and invalid), and check the format of the response returned.



## Write test for CRUD actions

For API specs, usually I will place the test (spec) files inside **spec/requests/<controller_name>/<action_name>.rb**, as RSpec recommended in their [directory structure](https://relishapp.com/rspec/rspec-rails/docs/directory-structure).



To test the creation function of bookmark, let’s create a test file at **spec/requests/bookmarks/create_spec.rb** .



We then automate the earlier manual actions, which is to create new bookmark with 

1. valid title and URL, and ensure we get a successful response, containing json of the created object.
2. invalid title or/and URL, and ensure we get an error message, and no new record is created



```ruby
# spec/requests/bookmarks/create_spec.rb

require 'rails_helper'

describe 'POST /bookmarks' do
  # 'scenario' is similar to 'it', use which you see fit
  
  scenario 'valid bookmark attributes' do
    # send a POST request to /bookmarks, with these parameters
    # The controller will treat them as JSON 
    post '/bookmarks', params: {
      bookmark: {
        url: 'https://rubyyagi.com',
        title: 'RubyYagi blog'
      }
    }

    # response should have HTTP Status 201 Created
    expect(response.status).to eq(201)

    json = JSON.parse(response.body).deep_symbolize_keys
    
    # check the value of the returned response hash
    expect(json[:url]).to eq('https://rubyyagi.com')
    expect(json[:title]).to eq('RubyYagi blog')

    # 1 new bookmark record is created
    expect(Bookmark.count).to eq(1)

    # Optionally, you can check the latest record data
    expect(Bookmark.last.title).to eq('RubyYagi blog')
  end

  scenario 'invalid bookmark attributes' do
    post '/bookmarks', params: {
      bookmark: {
        url: '',
        title: 'RubyYagi blog'
      }
    }

    # response should have HTTP Status 201 Created
    expect(response.status).to eq(422)

    json = JSON.parse(response.body).deep_symbolize_keys
    expect(json[:url]).to eq(["can't be blank"])

    # no new bookmark record is created
    expect(Bookmark.count).to eq(0)
  end
end

```



 Now run the test: `rspec spec/requests/bookmarks/create_spec.rb ` , and we will see two test cases pass, yay! 🎉



Now that we have a wrote test for the bookmark create action, we can move on to the update action.



If we want to update a bookmark, we would send a PATCH or PUT request to the **/bookmarks/:id** route. As we have created a bookmark earlier, which have the ID 1, we will send a PUT request to /bookmarks/1 .



![update bookmark API](https://rubyyagi.s3.amazonaws.com/14-rspec-request-spec/update_bookmark_manually.png)



Let’s create a new test file for the update action at **spec/requests/bookmarks/update_spec.rb**.



For the update action test, we would first need to create an existing bookmark record before we can edit it. We can create this bookmark record before the test is run, using **let!** helper method.



For example, `let!(:bookmark) { Bookmark.create(title: 'Ruby Yagi') }` will be executed before each “**scenario**” or “**it**” blocks, it will only execute once and its return value will be cached (subsequent call to it will always return the same value).



You can think of `let!(:bookmark) { Bookmark.create(title: 'Ruby Yagi') }` as a function like this :

```ruby
def bookmark
  Bookmark.create(title: 'Ruby Yagi')
end
```



then you can use **bookmark** method in your test code to access the created bookmark record. Jason Swett’s [article on let, let! and instance variables](https://www.codewithjason.com/difference-let-let-instance-variables-rspec/) has explained this in more detail.



```ruby
# spec/requests/bookmarks/update_spec

require 'rails_helper'

describe 'PUT /bookmarks' do
  # this will create a 'bookmark' method, which return the created bookmark object, 
  # before each scenario is ran
  let!(:bookmark) { Bookmark.create(url: 'https://rubyyagi.com', title: 'Ruby Yagi') }

  scenario 'valid bookmark attributes' do
    # send put request to /bookmarks/:id
    put "/bookmarks/#{bookmark.id}", params: {
      bookmark: {
        url: 'https://fluffy.es',
        title: 'Fluffy'
      }
    }

    # response should have HTTP Status 200 OK
    expect(response.status).to eq(200)

    # response should contain JSON of the updated object
    json = JSON.parse(response.body).deep_symbolize_keys
    expect(json[:url]).to eq('https://fluffy.es')
    expect(json[:title]).to eq('Fluffy')

    # The bookmark title and url should be updated
    expect(bookmark.reload.title).to eq('Fluffy')
    expect(bookmark.reload.url).to eq('https://fluffy.es')
  end

  scenario 'invalid bookmark attributes' do
    # send put request to /bookmarks/:id
    put "/bookmarks/#{bookmark.id}", params: {
      bookmark: {
        url: '',
        title: 'Fluffy'
      }
    }

    # response should have HTTP Status 422 Unprocessable entity
    expect(response.status).to eq(422)

    # response should contain error message
    json = JSON.parse(response.body).deep_symbolize_keys
    expect(json[:url]).to eq(["can't be blank"])

    # The bookmark title and url remain unchanged
    expect(bookmark.reload.title).to eq('Ruby Yagi')
    expect(bookmark.reload.url).to eq('https://rubyyagi.com')
  end
end

```



The [**reload**](https://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-reload) method will ask the bookmark (object) to query the database and get its latest value, instead of using the values stored in memory (which we did it in the let! helper method).



At this point, you know the basics of writing request spec that involves creating and updating a record. I will leave writing test for the delete action and list (index) action for you to complete on your own, **try writing your own test!** Reading a guided tutorial helps but to reinforce the knowledge you have learned in memory, you have to do type the code on your own.



To run all the test files at once, you can tell rspec to run all files inside the **spec/requests** folder like this :

`rspec spec/requests` .



## Testing authentication

If your Rails app has exposed API for consumption, most likely it will require authentication like API key, or user token to prevent abuse and also track usage for different users.



In this section we will update the tests we wrote earlier to include authentication checks.



Let's create a User model which contains username and authentication_token attributes, 

`rails g model user username:string authentication_token:string     `



Then run database migration to create the user table :

`rake db:migrate`



Then in the **bookmarks_controller.rb** file, we will add a method to validate the request based on the username and authentication_token received on the request.

```ruby
# bookmarks_controller.rb
class BookmarksController < ApplicationController
  before_action :authenticate_user
  
  # ....
  
  private
    def authenticate_user
      # find the user based on the headers from HTTP request
      @current_user = User.find_by(
        username: request.headers['X-Username'],
        authentication_token: request.headers['X-Token']
      )
      
      # return error message with 403 HTTP status if there's no such user
      return render(json: { message: 'Invalid User' }, status: 403) unless @current_user
    end
end
```



Now if we send a HTTP request without authentication headers, we will get an error :

![forbidden](https://rubyyagi.s3.amazonaws.com/14-rspec-request-spec/forbidden.png)



Let's create a user on the rails console (with username and authentication token), and try again :

![success auth](https://rubyyagi.s3.amazonaws.com/14-rspec-request-spec/success_auth.png)





Alright now we have an authentication mechanism in place, let's modify our previous tests to include these authentication details.



For **spec/requests/bookmarks/create_spec.rb** :

```ruby
# spec/requests/bookmarks/create_spec.rb

require 'rails_helper'

describe 'POST /bookmarks' do
  # create a user before the test scenarios are run
  let!(:user) { User.create(username: 'soulchild', authentication_token: 'abcdef') }

  # pass the user username and authentication to the header
  scenario 'valid bookmark attributes' do
    post '/bookmarks', params: {
      bookmark: {
        url: 'https://rubyyagi.com',
        title: 'RubyYagi blog'
      }
    }, headers: { 'X-Username': user.username, 'X-Token': user.authentication_token }
  # ...
  end

  # pass the user username and authentication to the header
  scenario 'invalid bookmark attributes' do
    post '/bookmarks', params: {
      bookmark: {
        url: '',
        title: 'RubyYagi blog'
      }
    }, headers: { 'X-Username': user.username, 'X-Token': user.authentication_token }
  # ...
  end
end
```



And for **spec/requests/bookmarks/update_spec.rb** :

```ruby
require 'rails_helper'

describe 'PUT /bookmarks' do
  let!(:bookmark) { Bookmark.create(url: 'https://rubyyagi.com', title: 'Ruby Yagi') }

  # create a user before the test scenarios are run
  let!(:user) { User.create(username: 'soulchild', authentication_token: 'abcdef') }

  scenario 'valid bookmark attributes' do
    # send put request to /bookmarks/:id
    # pass the user username and authentication to the header
    put "/bookmarks/#{bookmark.id}", params: {
      bookmark: {
        url: 'https://fluffy.es',
        title: 'Fluffy'
      }
    }, headers: { 'X-Username': user.username, 'X-Token': user.authentication_token }

    # ...
  end

  scenario 'invalid bookmark attributes' do
    # send put request to /bookmarks/:id
    # pass the user username and authentication to the header
    put "/bookmarks/#{bookmark.id}", params: {
      bookmark: {
        url: '',
        title: 'Fluffy'
      }
    }, headers: { 'X-Username': user.username, 'X-Token': user.authentication_token }

    # ...
  end
end
```



Before the test scenarios is run, we can create  a 'user' method using let!, which will returned the cache value of user.

```ruby
let!(:user) { User.create(username: 'soulchild', authentication_token: 'abcdef') }
```



And then we will use the user's username and authentication token in the request headers of the HTTP requests : 

```ruby
post '/bookmarks', params: {
  bookmark: {
    url: '',
    title: 'RubyYagi blog'
  }
}, headers: { 'X-Username': user.username, 'X-Token': user.authentication_token }
```



Run the test again `rspec spec/requests/bookmarks/create_spec.rb` (and update_spec.rb), the tests should pass.



Next, we are going to write one more scenario to test the response returned when there is no authentication header provided, it should return HTTP status 403 forbidden.



Before that, let's group the previous scenarios with valid authentication into a context for better readability  :

```ruby
# spec/requests/bookmarks/create_spec.rb
require 'rails_helper'

describe 'POST /bookmarks' do
  
  # group scenarios with authenticated user into this context block
  context 'authenticated user' do
    # create a user before the test scenarios are run
    let!(:user) { User.create(username: 'soulchild', authentication_token: 'abcdef') }

    # pass the user username and authentication to the header
    scenario 'valid bookmark attributes' do
      # .....
    end

    # pass the user username and authentication to the header
    scenario 'invalid bookmark attributes' do
      # ....
    end
  end
  
  # scenario with unauthenticated user
  context 'unauthenticated user' do
    # we will talk about this next.
  end
end

```



Alright, now we can write test for when the HTTP request doesn't contain the authentication headers information, it should return a response with HTTP status 403 (Forbidden), and a message saying 'invalid user'.



```ruby
# spec/requests/bookmarks/create_spec.rb
require 'rails_helper'

describe 'POST /bookmarks' do
  
  # group scenarios with authenticated user into this context block
  context 'authenticated user' do
    # create a user before the test scenarios are run
    let!(:user) { User.create(username: 'soulchild', authentication_token: 'abcdef') }

    # pass the user username and authentication to the header
    scenario 'valid bookmark attributes' do
      # .....
    end

    # pass the user username and authentication to the header
    scenario 'invalid bookmark attributes' do
      # ....
    end
  end
  
  # scenario with unauthenticated user
  context 'unauthenticated user' do
    it 'should return forbidden error' do
      post '/bookmarks', params: {
        bookmark: {
          url: 'https://rubyyagi.com',
          title: 'RubyYagi blog'
        }
      }

      # response should have HTTP Status 403 Forbidden
      expect(response.status).to eq(403)
      
      # response contain error message
      json = JSON.parse(response.body).deep_symbolize_keys
      expect(json[:message]).to eq('Invalid User')
    end
  end
end
```



You can write similar test with unauthenticated user on **update_spec.rb** as well. 



Most likely your API will return the same HTTP 403 response with identical error message across different endpoints, you might find yourself writing the same unauthenticated request test scenario (ie. response return 403 and same error message) across different API spec. To reduce repetition, you can look into [shared_examples](https://relishapp.com/rspec/rspec-core/docs/example-groups/shared-examples) . (I will write a tutorial on this in the future)

<script async data-uid="2f54587a09" src="https://rubyyagi.ck.page/2f54587a09/index.js"></script>


## Reference

[RSpec Request Spec documentation](https://relishapp.com/rspec/rspec-rails/docs/request-specs/request-spec)


