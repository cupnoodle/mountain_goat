---
layout: post
title: How to run tests in parallel in Github Actions
date: 2021-01-26 01:22 +0800
---

# How to run tests in parallel in Github Actions

The company I work for has recently switched to Github Actions for CI, as they offer 2000 minutes per month which is quite adequate for our volume. We have been running tests in parallel (2 instances) on the previous CI to save time, and we wanted to do the same on Github Actions.



Github Actions provides [strategy matrix](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix) which lets you run tests in parallel, with different configuration for each matrix.




For example with a matrix strategy below : 

```yml
jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        ruby:
          - '2.5.x'
          - '2.6.x'
          - '2.7.x'
        activerecord:
          - '5.0'
          - '6.0'
          - '6.1'
```

It will run 9 tests in parallel, 3 versions of Ruby x 3 versions of ActiveRecord = 9 tests

![9 tests](https://rubyyagi.s3.amazonaws.com/19-parallel-test-github-actions/9tests.png)







One downside of Github Actions is that they don't have built-in split tests function to split test files across different CI instances, which many other CI have. For example CircleCI provides split testing command like this : 

```bash
# Run rspec in parallel, and split the tests by timings
- run:
    command: |
      TESTFILES=$(circleci tests glob "spec/**/*_spec.rb" | circleci tests split --split-by=timings)
      bundle exec rspec $TESTFILES
```



With split testing, we can run different test files on different CI instances to reduce the test execution time on each instance.



Fortunately, we can write our own script to split the test files in Ruby.



Before we proceed to writing split test script, lets configure the Github Actions workflow to run our tests in parallel.



# Configuring Github Actions workflow to run test in parallel

Let's configure our Github Actions workflow to run test in parallel :



```yaml
# .github/workflows/test.yml
name: test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        # Set N number of parallel jobs you want to run tests on.
        # Use higher number if you have slow tests to split them on more parallel jobs.
        # Remember to update ci_node_index below to 0..N-1
        ci_node_total: [2]
        # set N-1 indexes for parallel jobs
        # When you run 2 parallel jobs then first job will have index 0, the second job will have index 1 etc
        ci_node_index: [0, 1]
    env:
      BUNDLE_JOBS: 2
      BUNDLE_RETRY: 3
      BUNDLE_PATH: vendor/bundle
      PGHOST: 127.0.0.1
      PGUSER: postgres
      PGPASSWORD: postgres
      RAILS_ENV: test
    services:
      postgres:
        image: postgres:9.6-alpine
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        env: 
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: bookmarker_test

    steps:
    # ...
```



The **ci_node_total** means the total number of parallel instances you want to spin up during the CI process. We are using 2 instances here so the value is [2]. If you would like to use more instances, say 4 instances, then you can change the value to [4] : 

```yaml
ci_node_total: [4]
```



The **ci_node_index** means the index of the parallel instances you spin up during the CI process, this should match the ci_node_total you have defined earlier.



For example, if you have 2 total nodes, your **ci_node_index** should be [0, 1]. If you have 4 total nodes, your **ci_node_index** should be [0, 1, 2, 3]. This is useful for when we write the script to split the tests later.



The **fail-fast: false** means that we want to continue running the test on other instances even if there is failing test on one of the instance. The default value for fail-fast is true if we didn't set it, which will stop all instances if there is even one failing test on one instance. Setting fail-fast: false would allow all instances to finish run their tests, and we can see which test has failed later on.



Next, we add the testing steps in the workflow : 

{% raw %}
```yaml
steps:
  # ... Ruby / Database setup...
  # ...
  - name: Make bin/ci executable
    run: chmod +x ./bin/ci
  - name: Run Rspec test
    id: rspec-test
    env:
      # Specifies how many jobs you would like to run in parallel,
      # used for partitioning
      CI_NODE_TOTAL: ${{ matrix.ci_node_total }}
      # Use the index from matrix as an environment variable
      CI_NODE_INDEX: ${{ matrix.ci_node_index }}
    continue-on-error: true
    run : |
      ./bin/ci
```
{% endraw %}


We pass the **ci_node_total** and **ci_node_index** variables into the environment variables (**env:** ) for the test step.



The **continue-on-error: true** will instruct the workflow to continue to the next step even if there is error on the test step (ie. when any of the test fails).



And lastly we want to run the custom test script that split the test files and run them  **./bin/ci** , remember to make this file executable in the previous step.



We will look into how to write the **bin/ci** in the next section.





## Preparing the ruby script to split test files and run them

In this section, we will write the **bin/ci** file.

First, create a file named "**ci**" (without file extension), and place it in the "**bin**" folder inside your Rails app like this :
![ci_file_location](https://rubyyagi.s3.amazonaws.com/19-parallel-test-github-actions/ci_file.png)



Here's the code for the **ci** file :

```ruby
#!/usr/bin/env ruby

tests = Dir["spec/**/*_spec.rb"].
  sort.
  # Add randomization seed based on SHA of each commit
  shuffle(random: Random.new(ENV['GITHUB_SHA'].to_i(16))).
  select.
  with_index do |el, i|
    i % ENV["CI_NODE_TOTAL"].to_i == ENV["CI_NODE_INDEX"].to_i
  end

exec "bundle exec rspec #{tests.join(" ")}"
```



`Dir["spec/**/*_spec.rb"]` will return an array of path of test files, which are located inside the **spec** folder , and filename ends with **_spec.rb** . (eg: *["spec/system/bookmarks/create_spec.rb", "spec/system/bookmarks/update_spec.rb"]*) . 

You can change this to `Dir["test/**/*_test.rb"]` if you are using minitest instead of Rspec.



**sort** will sort the array of path of test files in alphabetical order.



**shuffle** will then randomize the order of the array, using the seed provided in the **random** parameter, so that the randomized order will differ based on the **random** parameter we passed in.



**ENV['GITHUB_SHA']** is the SHA hash of the commit that triggered the Github Actions workflow, which available as environment variables in Github Actions.



As **Random.new** only accept integer for the seed parameter, we have to convert the SHA hash (in hexadecimal string, eg: fe5300b3733d69f0a187a5f3237eb05bb134e341 ) into integer using hexadecimal as base (**.to_i(16)**).



Then we select the test files that will be run using **select.with_index**. For each CI instance, we will select the  test files from the array, which the remainder of the index divided by total number of nodes equals to the CI instance index.



For example, say CI_NODE_TOTAL = 2, CI_NODE_INDEX = 1, and we have four test files :

![explanation](https://rubyyagi.s3.amazonaws.com/19-parallel-test-github-actions/explanation.png)

The test files will be split as shown above.



The downside of this split is that the tests will not be splitted based on the duration of the test, which might make some instance running longer than the others. CircleCI does store historical data of each tests duration and can split the test files based on timing, hopefully Github Actions will implement this feature in the future.



## Example repository

I have created an example Rails repository with parallel tests for Github Actions here : [https://github.com/rubyyagi/bookmarker](https://github.com/rubyyagi/bookmarker)



You can check out the [full workflow .yml file here](https://github.com/rubyyagi/bookmarker/blob/main/.github/workflows/test.yml) , the [bin/ci file here](https://github.com/rubyyagi/bookmarker/blob/main/bin/ci) , and an [example test run here](https://github.com/rubyyagi/bookmarker/actions/runs/507483357).



<script async data-uid="4776ba93ea" src="https://rubyyagi.ck.page/4776ba93ea/index.js"></script>