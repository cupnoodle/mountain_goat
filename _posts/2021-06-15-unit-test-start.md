---
layout: post
title: Getting started with testing mindset and CI on Ruby
date: 2021-06-15 19:06 +0800
---

If you have been writing Ruby / Rails code for a while, but still can't grasp on the '**why**' to write automated test, (eg. why do so many Ruby dev job posting requires automated test skill like Rspec, minitest etc? How does writing automated test makes me a better developer or help the company I work with? ) 



or if you are not sure on how the **CI** (continuous integration) stuff works, this article is written to address these questions!



This article won't go into how to write unit test, but it will let you **experience what is it like working on a codebase that has automated testing in place, and how it can speed up the development flow and assure quality** (ie. confirm the code works). This article assume you have basic knowledge on how to use Git and Github.



I have written a small Ruby Sinatra app (with automated test written using Rspec, you dont have to worry about Rspec for now), you can fork it to your Github account: [https://github.com/rubyyagi/cart](https://github.com/rubyyagi/cart) , then follow along this article to add a feature for it.



![fork](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/fork.png)





After forking the repo, you can download the repo as zip or clone it to your computer. The repo is a simple shopping cart web app, which user can add items to cart and checkout, you can also try the completed online demo here : [https://alpine-cart.herokuapp.com](https://alpine-cart.herokuapp.com)



I recommend **creating a new git branch** before start working on it, to make creating pull request easier in the next step. (create a new branch instead of working on master branch directly.)


Remember to run `bundle install` after downloading the repo, to install the required gem (Sinatra and Rspec for testing).



To run the web app, open terminal, navigate to that repo, and run `ruby app.rb` , it should spin up a Sinatra web app, which you can access in your web browser at "**localhost:4567**"




You can add items to the cart, then click '**Checkout**' , which it then will show the total (you will implement the discount part later).



![checkout](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/example.gif)



Open the repo with your favorite code editor, you will notice a folder named "**spec**" :

![specs](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/specs.png)



This folder contains the test script I have pre written, there are two test files inside (bulk_discount_spec.rb and cart_spec.rb) , they are used to verify if the **bulk_discount** logic and **cart** logic is implemented correctly, you don't have to make changes on these files.



To run these test, open terminal, navigate to this repo, and run `rspec` (type 'rspec' and press enter).

You should see some test cases fail like this : 

![test fail](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/fail_test.png)



This is expected, as the **bulk_discount.rb** file (it contains logic for bulk discount, located in  models/bulk_discount.rb) has not been implemented yet, hence the total returned from the cart on those test are incorrect.




For this exercise, you will need to implement the bulk discount logic in **models/bulk_discount.rb** , specifically inside the **def apply** method.

![bulk discount file](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/bulk_discount.png)



The **BulkDiscount** class has three properties, amount (in cents), quantity_required (integer), and product_name (string).



For example in the **app.rb**, we have two bulk discounts :

```ruby
discounts = [
  BulkDiscount.new(amount: 100, quantity_required: 3, product_name: 'apple'),
  BulkDiscount.new(amount: 200, quantity_required: 2, product_name: 'banana')
]

cart = Cart.new(discounts: discounts, products: products)
```



We want to give $1 (100 cents in amount) discount off the cart for every 3 Apple (product_name: 'apple') purchased. For example if the customer buys 7 Apple, the cart will be discounted $2  ($1 discount for every 3 apple, 6 apple = discount $2).


For the code in **app.rb** , we want to give bulk discount for apple and banana.  Say if the cart has 7 apple and 5 banana, it will be discounted with $6  ($1 x 2 + $2 x 2 = $6, discount of $1 x 2 for apple, $2 x 2 for banana).



Back to **bulk_discount.rb**,  the **apply** method accepts two parameter, first parameter **total** , which is the total of the carts, in cents (eg: $4 is 400 cents), before the current bulk discount is applied. 



The second parameter **products** in an array of product (you can check **models/product.rb**) that is currently in the cart.



The **apply** method should go through all of the product in the products array, and check if there is product matching the **product_name** of the BulkDiscount, and then check if the quantity of the product is larger than the **quantity_required** , and calculate the amount to be deducted and save it to **amount_to_deduct**.



Then the **apply** method will return the discounted total (total - amount_to_deduct).



Each time you finish writing the code, you can run `rspec` to check if the test cases pass. If all of them pass, it means your implementation is correct.



This feedback loop process should be a lot faster than opening the web browser and navigate to the web app, add a few products in cart, click 'checkout' and see if the discount is displayed and if the displayed discount amount is correct.

![rspec](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/rspec.gif)



Compared to opening web browser and press 'add to cart' and click 'checkout' every time you want to test if your code is correct :



![example](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/example.gif) 



With automated test, you can check if an implementation is correct a lot faster than testing it manually.



And with automated test in place, you can be confident that you didn't break existing feature when you add new feature or make changes in the future, which makes refactoring safe.



(Please work on the exercise yourself first before checking the answer 🙈)



You can compare your implementation with my implementation here ([https://github.com/rubyyagi/cart/blob/answer/models/bulk_discount.rb](https://github.com/rubyyagi/cart/blob/answer/models/bulk_discount.rb)) for the bulk discount.





## Continuous Integration (CI)

Once you have finished implementing the bulk discount code part and ensure the test passes, you can push the commit to your forked repo.




And then you can create a new pull request from your branch to the original repo (the **rubyyagi/cart**) like this  :



![pull request](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/pr1.png)



![pull request 2](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/pr2.png)



After creating the pull request, you will notice that Github (Github Actions) automatically runs the test (same like when you run `rspec`) :

![CI](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/ci1.png)


Once the test has passed, it will show this :

![CI 2](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/ci2.png)



Or if there is failing test, it will show "fail".


The idea is that when you submit a new pull request (containing new feature or fixes), an automated test suite will be run on some external service (Github Actions for this example, but there's also other CI service provider like CircleCI, SemaphoreCI etc). And you can configure the repository such that only pull request with passing test can be merged.



You can take a look at [pull requests in the official Rails repository](https://github.com/rails/rails/pulls), each pull request will trigger an automated test run, to ensure they don't break existing Rails feature, this inspires confidence on the overall code quality.

![ci3](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/ci3.png)



In my understanding, **Continuous Integration** (CI) is the process of 

1. Create new branch to work on new feature or fixes
2. Create pull request from the new branch to the main / master branch
3. Running automated test suite (and also linting, security checks) on the branch to ensure everything works
4. A senior dev / team lead/ repository owner will code review the new branch, and request changes if required, else they can approve the new branch and merge it to master/ main branch
5. The new branch is merged to master/ main branch
6. The master / main branch is deployed (to Heroku or AWS or some other service) manually or automatically, or if it is a library , push to RubyGems.org



If you are using Heroku, you can configure to automatic deploy new merge to master / main branch only if the CI (automated test suite) passes :



![cd](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/cd1.png)

![cd2](https://rubyyagi.s3.amazonaws.com/22-unit-test-start/cd2.png)



I have already setup the automated test suite to run on Github actions for this repository, you can check the workflow file here if you are interested : [https://github.com/rubyyagi/cart/blob/master/.github/workflows/test.yml](https://github.com/rubyyagi/cart/blob/master/.github/workflows/test.yml) . 



Different CI usually requires different configuration file on the repository, eg: for CircleCI, you need to write the configuration file in `/.circleci/config.yml` in your repository, these config are vendor specific.



Hope this article has made you understand the importance of automated testing, and how it can speed up development process and inspire confidence in code quality!




## Further Reading

[Introduction to Rails testing with RSpec and Capybara](https://rubyyagi.com/intro-rspec-capybara-testing/)

[Introduction to API testing using RSpec and Request Spec](https://rubyyagi.com/rspec-request-spec/)



[The beginner guide to Rails testing, written by Jason Swett](https://www.codewithjason.com/rails-testing-guide/)


<script async data-uid="4776ba93ea" src="https://rubyyagi.ck.page/4776ba93ea/index.js"></script>