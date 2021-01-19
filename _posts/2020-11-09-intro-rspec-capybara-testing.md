---
layout: post
title: Introduction to Rails testing with RSpec and Capybara
---

Testing is heavily emphasized in the Ruby and Rails community, even the official Ruby on Rails guide [has a section dedicated to testing](https://guides.rubyonrails.org/testing.html). A lot of Rails job postings usually requires testing skill (Rspec, Minitest etc) as well : 

![rails job requirement](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/rails_job_desc.png)



I didn't understand the importance of testing and used to think writing test was an extra effort (my app already work well when I wrote it, why should I spend extra time to write test for it?), boy I was so wrong 😅. 



Now I work in an enterprise handling thousands of transaction per day, I would be very afraid to deploy new update if there wasn't a solid test suite, without a test suite, I could have broke the payment system on a new deploy, which could cost the companies a lot of $$$ 💸 ! This is why many companies with good engineering practice would require job candidate to possess skill on writing automated test.



Automated test can make us confident in our deploy, ensure there is no regression (ie. new deploy broke old working features), and in contrary, it can make the development process faster too! Instead of filing forms and clicking buttons manually every time you make a change (10 seconds), we can tell Capybara to do this for us (in 1-2 seconds).



This article aims to give you a guideline on how to get started on testing, be it on your newly created Rails app or an existing Rails app that already has some users. We will first write a test to check static content, then move to testing basic CRUD feature.



We will be using **Rspec** (the testing framework) and **Capybara** (Browser automation tool to fill in text field, click button) in this article. This article will start from a freshly created Rails app, but you can follow the same instruction on your existing Rails app as well.

**Table of contents**
1. [Creating the sample Rails app](#creating-the-sample-rails-app)
2. [Install Rspec-rails and Capybara](#install-rspec-rails-and-capybara)
3. [Create a static controller displaying static content](#create-a-static-controller-displaying-static-content)
4. [Write your first test](#write-your-first-test)
5. [Ensure the test actually work by making it fail](#ensure-the-test-actually-work-by-making-it-fail)
6. [Create a CRUD Scaffold](#create-a-crud-scaffold)
7. [Write test for CRUD actions](#write-test-for-crud-actions)
8. [No idea on how to start writing automated test?](#no-idea-on-how-to-start-writing-automated-test)
9. [References](#references)


## Creating the sample Rails app

First let's create a Rails app using **rails new** :

`rails new bookmarker -T`



 You can use any app name you want,  the `-T` flag is used to tell Rails not to use MiniTest. By default, Rails will use a test framework called 'MiniTest' if we didn't specify `-T` flag, we want to use **Rspec** as the test framework instead of MiniTest.



Next, create the database of the Rails app so that we can run the app later :

```ruby
cd bookmarker
rake db:create
```



## Install Rspec-rails and Capybara

Now, we will install the rspec-rails, capybara and webdrivers gems into our Rails app.



**rspec-rails** is the rspec testing framework, plus some adopted Rails magic. For example when you run `rails generate model User` to create user , it will also automatically create a model test file for that user : **user_spec.rb**  .



**capybara** allows us to interact with web browser using ruby code, eg: `click_link 'Go to next page'` .



**webdrivers** is responsible for downloading the driver needed to control the web browser (eg: chromedriver), and we will use this driver to launch and run automated action on the web browser.



Include these 3 gems in the **Gemfile**, inside the **:development, :test** and **:test** group like this :

```ruby
# Gemfile

# ...
group :development, :test do
  gem 'rspec-rails'
end

group :test do
  gem 'capybara'
  gem 'webdrivers', '~> 4.0', require: false
end
```

We don't need these gem in production, grouping them in development/test group will exclude these gem from installing in production, thus reducing the application size.



If you are using an existing Rails app, the `capybara` and `webdrivers`  gem might be already included in the **:test** group, then you only need to add `rspec-rails` to the **:development, :test** group.



Then run `bundle install` to install them.



After installing these gems, we still need to install Rspec into our app using 

`rails generate rspec:install`  .

This will create a few boilerplate files, and update some configuration in our Rails app.

Remember to require the webdrivers gem at the top of **spec/rails_helper.rb** :
```ruby
# spec/rails_helper.rb
# This file is copied to spec/ when you run 'rails generate rspec:install'
require 'spec_helper'
require 'webdrivers'
```


## Create a static controller displaying static content

In real life applications, you probably won't need to write a test just to verify static content, this section serve just as a starting point on writing test.



In this section we will create a page displaying static text, then write a test to verify if the text displayed is exactly what we want.



Let's create a controller **StaticController** with just a single action **index** : 

`rails generate controller static index`



This will create **static_controller.rb** and **static/index.html.erb** . Next, open static/index.html.erb, and add a static text, say 'Hello world' into it :

```html
<!-- static/index.html.erb -->
<h1>Static#index</h1>
<p>Find me in app/views/static/index.html.erb</p>
<h2>
Hello world
</h2>
```



Start the rails server using `rails server` , then navigate to **localhost:3000/static/index**, you should see the page output like this :

![static page](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/static_hw.png)

Now that we have created a static page, let's write a test for it.



## Write your first test

Let's create a file named "**static_spec.rb**" , and place it in "**spec/static_spec.rb**" : 

```ruby
# spec/static_spec.rb

require 'rails_helper'

RSpec.describe 'Static content', type: :system do
  it 'shows the static text' do
    visit static_index_path
    expect(page).to have_content('Hello world')
  end
end
```



Then in your terminal, navigate to your Rails app root path, then run `rspec spec/static_spec.rb`



You should see Chrome open and it will load the page briefly and close, and a succeeded test : 

![succeed](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/succeed.png)



Congratulation! You have written your first Rspec test (if you haven't already before 😬) . The convention of RSpec is that all test files must end with "_spec" in their filename.



Below is the explanation of what each line does in the test file.



```ruby
require 'rails_helper'
```

This line will load the necessary test configuration from **spec/rails_helper.rb**, which is needed for the test to run.



```ruby
RSpec.describe 'Static content', type: :system do
```

This line is how we start writing the test, the test code usually reside inside the describe block.



You can replace 'Static content' with any string you want, it is for your own documentation purpose.



The **type: :system** will tell Rspec that this is a system test, which Rspec will then load Capybara and use the browser to run it.



```ruby
it 'shows the static text' do
```

This line is also for documentation purpose, you replace the string with any string you want, but note that the **expect()** assertion will only works inside a **it** or **scenario** block.



```ruby
visit static_index_path
```

This will tell the browser to navigate to the static_index_path, which the static_index_path is just a Rails routing helper method. **visit** is a method from Capybara.



The above is equivalent to `visit /static/index` .



```
expect(page).to have_content('Hello world')
```

This will tell Rspec to ensure the current page loaded (the special **page** variable), has the text '**Hello world**'. The [**have_content**](https://www.rubydoc.info/github/jnicklas/capybara/Capybara%2FRSpecMatchers:have_content) method here is provided by Capybara, as it is browser related.

 

The above syntax might look magical at first, it is equivalent to 

```ruby
expect(page).to(have_content('Hello world'))
```



With ruby magic, we can omit the parentheses for the **to** method.



## Ensure the test actually work by making it fail

How do we know our test actually work? What if the test we wrote always passes because we didn't check for enough condition? This might render the test useless as it doesn't guard us from breaking the feature.



To ensure the test actually work, we can try break it on purpose. As our test check for the text 'Hello world', we can modify the view to display a different text, say 'Bonjour world'.

```html
<!-- static/index.html.erb -->
<h1>Static#index</h1>
<p>Find me in app/views/static/index.html.erb</p>
<h2>
Bonjour world
</h2>
```



Then run the same test again : 

` rspec spec/static_spec.rb`



Sure enough, the test fails :

![failing test](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/failure.png)



If you are wondering where does the "Static content shows the static text" string comes from, it is from the strings you used in the **describe** and **it** blocks :

![failing test documentation](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/failure_document.png)



The strings you used in the **describe** and **it**  block will serve as documentation when your test goes wrong, to help you pinpoint which part of your test has failed.



The terminal also shows a path to the screenshot for the failed case, which can be helpful to debug why a certain test fail, especially dealing with AJAX calls.



## Create a CRUD scaffold

Next, we are going to create scaffold (model + controller + view) so that we can write test for basic CRUD features (create, read, update, delete). Usually scaffold will work out of the box without needing to test it, nevertheless in this section I will try explain my experience / thought on how to test CRUD.



Let's create a sample scaffold, I used "Bookmark", with two string properties : "title" and "url".

`rails generate scaffold Bookmark title:string url:string`



As we don't want user to input empty values into title and url column, open up **db/migrate/...._create_bookmarks.rb** and add '**null: false**' to them



```ruby
# 20201104084809_create_bookmarks.rb
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

`rake db:migrate`



Next, open the bookmark model file in **app/models/bookmark.rb** , and add validation for presence (not null), and disallow blank value : 

```ruby
# app/models/bookmark.rb
class Bookmark < ApplicationRecord
  validates :title, presence: true, allow_blank: false
  validates :url, presence: true, allow_blank: false
end
```



Run `rails server` and navigate to **localhost:3000/bookmarks**, we can see that Rails already generated views for creating, editing and deleting bookmark model for us : 

![bookmarks index](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/bookmarks_index.png)



If we try to create a bookmark with blank title and url, it will fail with error message :

![blank bookmark](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/blank_bookmark.png)



We will get an error message saying the attributes can't be blank, and no bookmark record will be created in the database, we can proceed to automate checking these in the next section.



## Write test for CRUD actions

For CRUD actions, usually I will place the test (spec) files inside **spec/system/\<controller_name\>**, as RSpec recommended in their [directory structure](https://relishapp.com/rspec/rspec-rails/docs/directory-structure). They are called system spec as it allow us to test user interaction using a browser (which uses Capybara under the hood) : 

> System tests allow you to test user interactions with your application,
> running tests in either a real or a headless browser. System tests use
> Capybara under the hood.



To test the creation function of bookmark, let's create a test file at **spec/system/bookmark/create_spec.rb** .



We then automate the earlier manual test, which is to create new bookmark with empty title and URL, then ensure we get an error message and no new record is created.

```ruby
# spec/system/bookmark/create_spec.rb

require 'rails_helper'

RSpec.describe 'create bookmark', type: :system do
  scenario 'empty title and url' do
    visit new_bookmark_path
    click_button 'Create Bookmark'

    # The page should show error message 'Title can't be blank'
    # 'page' is a special variable from capybara, which contain information of the current displayed page
    expect(page).to have_content("Title can't be blank")

    # No bookmark record is created
    expect(Bookmark.count).to eq(0)
  end
end
```



Then run the test using `rspec spec/system/create_spec.rb` , we should see the test pass, yay!



Next, we can write another scenario for the happy path which will create the bookmark successfully.



If we create a bookmark manually, we would see a success message like this : 

![new happy path](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/new_happy_path.png)



![successfully create new bookmark](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/success.png)



Let's automate this manual test into automated test.



```ruby
# spec/system/bookmark/create_spec.rb

require 'rails_helper'

RSpec.describe 'create bookmark', type: :system do
  scenario 'empty title and url' do
    visit new_bookmark_path
    click_button 'Create Bookmark'

    # The page should show error message 'Title can't be blank'
    expect(page).to have_content("Title can't be blank")

    # No bookmark record is created
    expect(Bookmark.count).to eq(0)
  end
  
  # happy path scenario block
  scenario 'valid title and url' do
    visit new_bookmark_path
    # fill in text fields with specified string
    fill_in 'Title', with: 'RubyYagi'
    fill_in 'Url', with: 'https://rubyyagi.com'
    click_button 'Create Bookmark'
    
    # The page should show success message
    expect(page).to have_content("Bookmark was successfully created")

    # 1 new bookmark record is created
    expect(Bookmark.count).to eq(1)
    
    # Optionally, you can check the latest record data
    expect(Bookmark.last.title).to eq('RubyYagi')
  end
end
```



We can use the [**fill_in**](https://rubydoc.info/github/jnicklas/capybara/Capybara/Node/Actions:fill_in) method from Capybara to fill in text into the Title and URL text field. We can pass the label of the text field, or name of the input, or id of the input as the first parameter.



![fill in parameters](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/fill_in_parameters.png)





Now run the test again : `rspec spec/system/create_spec.rb `  , and we will see two test cases pass.



Now that we have a wrote test for the bookmark create action, we can move on to the update action.



If we want to update a bookmark, we would go to the bookmarks index page, click 'Edit' on the bookmark we want to edit, change the title or URL, and press ""



![list_page](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/list_page.png)



![edit page](https://rubyyagi.s3.amazonaws.com/12-intro-rspec-capybara/edit_page.png)



Let's create a new test file for the update action at **spec/system/bookmark/update_spec.rb**.



For the update action test, we would first need to create an existing bookmark record before we can edit it. We can create this bookmark record before the test is run, using **let!** helper method.



For example, `let!(:bookmark) { Bookmark.create(title: 'Ruby Yagi') }`  will be executed before each "**scenario**" or "**it**" blocks,  it will only execute once and its return value will be cached (subsequent call to it will always return the same value). 



You can think of `let!(:bookmark) { Bookmark.create(title: 'Ruby Yagi') }` as a function like this :

```ruby
def bookmark
  Bookmark.create(title: 'Ruby Yagi')
end
```



then you can use **bookmark** method in your test code to access the created bookmark record. Jason Swett's [article on let, let! and instance variables](https://www.codewithjason.com/difference-let-let-instance-variables-rspec/) has explained this in more detail.



```ruby
# spec/system/bookmark/update_spec.rb

require 'rails_helper'

RSpec.describe 'update bookmark', type: :system do
  # this will create a 'bookmark' method, before each scenario is ran
  let!(:bookmark) { Bookmark.create(url: 'https://rubyyagi.com', title: 'Ruby Yagi') }

  scenario 'empty title and url' do
    visit bookmarks_path
  
    # click the link that has the text 'Edit'
    click_link 'Edit'
    
    fill_in 'Title', with: ''
    fill_in 'Url', with: ''

    click_button 'Update Bookmark'

    # The page should show error message 'Title can't be blank'
    expect(page).to have_content("Title can't be blank")

    # The bookmark title should be unchanged
    expect(bookmark.reload.title).to eq('Ruby Yagi')
  end

  scenario 'valid title and url' do
    visit bookmarks_path

    click_link 'Edit'
    fill_in 'Title', with: 'Fluffy'
    fill_in 'Url', with: 'https://fluffy.es'

    click_button 'Update Bookmark'

    # The page should show success message
    expect(page).to have_content("Bookmark was successfully updated")

    # The bookmark title and url should be updated
    expect(bookmark.reload.title).to eq('Fluffy')
    expect(bookmark.reload.url).to eq('https://fluffy.es')
  end
end
```



The **click_link** is also a method from Capybara, it will tell the browser to click a link that has the text will specify.



The [**reload**](https://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-reload) method will ask the bookmark (object) to query the database and get its latest value, instead of using the values stored in memory (which we did it in the let! helper method).



At this point, you know the basics of writing test for create and updating a record. I will leave writing test for the delete action and list (index) action for you to complete on your own, **try writing your own test!** Reading a guided tutorial helps but to reinforce the knowledge you have learned in memory, you have to do type the code on your own.



To run all the test files at once, you can tell rspec to run all files inside the **spec/system** folder like this :

`rspec spec/system` .





## No idea on how to start writing automated test?

When I first learned writing automated test, I would often get stuck on how to start writing test for a new feature or controller / page I am implementing.



To get unstuck, I would open the web browser and go to the Rails app, run every action manually, like fill in text in this form, click this button, check if the database record is created, check the text that appear in the web browser etc. Then I note them down on a notebook using pencil, then I would try to type the equivalent Capybara or Ruby code for them in the test file.




You can start with manual test first (open web browser and type in text, click button, check result etc), as we are used to this when coding feature, then try to convert the manual action into automated test code.


<script async data-uid="2f54587a09" src="https://rubyyagi.ck.page/2f54587a09/index.js"></script>


## References

[Official Capybara documentation](https://rubydoc.info/github/teamcapybara/capybara#using-capybara-with-rspec)
