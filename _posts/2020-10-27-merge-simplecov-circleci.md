---
layout: post
title: How to merge SimpleCov coverage result on CircleCI 2.0 for parallel test
---

At my work company, we use [CircleCI](https://circleci.com) to run our tests on every commit, and recently we have been looking at running test in parallel using their parallelism feature (which split the tests to multiple container, so it runs faster).



We use [SimpleCov](https://github.com/simplecov-ruby/simplecov) gem to generate test coverage report, which shows us how much code we wrote are covered by our own test, so that we can spot which part of code is not tested thoroughly.



![sample coverage report](https://rubyyagi.s3.amazonaws.com/11-merge-simplecov-circleci/sample_coverage.png)





SimpleCov works great when running just on 1 VM, but when we run it on multiple VMs in parallel, the test files get splitted across different VM and the coverage result is not as accurate anymore. eg: Say you have 4 parallel VM, the test files get divided by 4, and the test coverage will become 1/4 of the original coverage.



Fortunately SimpleCov allow us to export coverage result in .json format on each parallel VMs, and collate (join) them together and generate the final report after all tests have been done. In this post I will explain the steps needed to implement this.



## Step 1 - Setup SimpleCov

Include simplecov in your **gemfile** (inside the test group)

```ruby
group :test do
  gem 'simplecov', require: false
end
```



Next, in your **spec/spec_helper.rb** file,  insert the code below in between all the `require` blocks at the top of the file :

```ruby
#spec_helper.rb

require 'simplecov'
SimpleCov.start 'rails' do
  # Disambiguates individual test runs with CIRCLE_NODE_INDEX
  command_name "Job #{ENV['CIRCLE_NODE_INDEX']}" if ENV['CIRCLE_NODE_INDEX']
  
  # If running test in CI, generate just .json result, then we can join them later
  # else, generate the full HTML report
  if ENV['CI']
    formatter SimpleCov::Formatter::SimpleFormatter
  else
    formatter SimpleCov::Formatter::MultiFormatter.new(
      [
        SimpleCov::Formatter::SimpleFormatter,
        SimpleCov::Formatter::HTMLFormatter
      ]
    )
  end

  track_files "**/*.rb"
end

require File.expand_path('../config/environment', __dir__)
require 'rspec/rails'
```



The `command_name "Job #{ENV['CIRCLE_NODE_INDEX']}" if ENV['CIRCLE_NODE_INDEX']` will tell simplecov the **name** of the test suite that is currently running.



**CIRCLE_NODE_INDEX** is an environment variable provided by CircleCI, say you are running a test with 4 parallelism, then the first container will have value '**0**' for CIRCLE_NODE_INDEx , the second container will have value '**1**', the third container will have value '**2**' and so on.



With the **command_name,** simplecov can distinguish different running test suite.



When running in CI, we tell SimpleCov to use SimpleFormatter, it will generate a .json containing coverage result from the test run, that is stored in **coverage/.resultset.json**. Later we will join all these .json to generate the final HTML report.



## Step 2 - Create rake task to join result json and generate HTML report

After the resultset.json is created in each parallel test, we need to join them together using **collate** to generate HTML report. 




Let's create a rake task for this, create a rake file in **lib/tasks/coverage_report.rake** :

```ruby
# lib/tasks/coverage_report.rake

namespace :coverage do
  task :report do
    require 'simplecov'

    SimpleCov.collate Dir["coverage_results/.resultset-*.json"], 'rails' do
      formatter SimpleCov::Formatter::MultiFormatter.new(
        [
          SimpleCov::Formatter::SimpleFormatter,
          SimpleCov::Formatter::HTMLFormatter
        ]
      )
    end
  end
end

```



The rake task above will join all .resultset-*.json (eg: .resultset-0.json, .resultset-1.json, .resultset-2.json etc) files inside coverage_results folder, then generate a HTML report from it.



## Step 3 - Add parallelism and code coverage generation to the CircleCI YML file

Here's the **.circleci/config.yml** I used (with some modification) :

```yml
---

version: 2

references:
  default_docker_ruby_executor: &default_docker_ruby_executor
    image: ruby:2.5
    environment:
      BUNDLE_JOBS: 2
      BUNDLE_RETRY: 3
      BUNDLE_PATH: vendor/bundle
      PGHOST: 127.0.0.1
      PGUSER: runner
      PGPASSWORD: ""
      RAILS_ENV: test
      REDIS_URL: redis://localhost:6379
      COVERAGE: true
  postgres: &postgres
    image: circleci/postgres:9.6-alpine
    environment:
      POSTGRES_USER: runner
      POSTGRES_PASSWORD: ""
      POSTGRES_DB: appname_test

jobs:
  test:
    parallelism: 2
    docker:
      - *default_docker_ruby_executor
      - *postgres
    steps:
      - checkout
      - restore_cache:
          keys:
            - appname-bundle-{{ checksum "Gemfile.lock" }}
            - appname-bundle-
      - run:
          name: Install Bundler Gem
          command: gem install bundler --no-ri --no-rdoc
      - run:
          name: Bundle Install
          command: bundle check || bundle install
      # Store bundle cache
      - save_cache:
          key: appname-bundle-{{ checksum "Gemfile.lock" }}
          paths:
            - vendor/bundle
      - restore_cache:
          keys:
            - appname-yarn-{{ checksum "yarn.lock" }}
            - appname-yarn-
      - run:
          name: Yarn Install
          command: yarn install --cache-folder ~/.cache/yarn
      - save_cache:
          key: appname-yarn-{{ checksum "yarn.lock" }}
          paths:
            - ~/.cache/yarn
      - run:
          name: Database setup
          command: RAILS_ENV=test bundle exec rake db:create db:schema:load

      # Run rspec in parallel
      - run:
          command: |
            mkdir /tmp/test-results
            TESTFILES=$(circleci tests glob "spec/**/*_spec.rb" | circleci tests split --split-by=timings)
            xvfb-run --auto-servernum bundle exec rspec $TESTFILES --profile 10 --color --format progress --format RspecJunitFormatter --out /tmp/test-results/rspec.xml
      - store_test_results:
          path: /tmp/test-results
      - run:
          name: Stash Coverage Results
          command: |
            mkdir coverage_results
            cp -R coverage/.resultset.json coverage_results/.resultset-${CIRCLE_NODE_INDEX}.json
      - persist_to_workspace:
          root: .
          paths:
            - coverage_results
      - store_artifacts:
          path: tmp/test_prof

  coverage:
    docker:
      - image: ruby:2.5
    environment:
      BUNDLE_JOBS: 2
      BUNDLE_RETRY: 3
      BUNDLE_PATH: vendor/bundle
    steps:
      - checkout
      - attach_workspace:
          at: .
      - restore_cache:
          keys:
            - appname-bundle-{{ checksum "Gemfile.lock" }}
            - appname-bundle-
      - run:
          name: Bundle Install
          command: bundle check || bundle install
      - run:
          name: Merge and check coverage
          command: |
            bundle exec rake coverage:report
      - store_artifacts:
          path: coverage

workflows:
  version: 2
  deployment:
    jobs:
      - test
      - coverage:
          requires:
            - test
```

 

After running test, we will copy the **.resultset json** to **coverage_results/.resultset-${CIRCLE_NODE_INDEX}.json** : 

```bash
cp -R coverage/.resultset.json coverage_results/.resultset-${CIRCLE_NODE_INDEX}.json
```



Then we save the **coverage_results** folder into the workspace, so that it can be used in the next workflow (coverage).

```yml
- persist_to_workspace:
    root: .
    paths:
      - coverage_results
```



We can store folder/files into workspace, and use it in another workflow later on. 


![coverage](https://rubyyagi.s3.amazonaws.com/11-merge-simplecov-circleci/workspace.png)



Then in the coverage workflow, we attach the workspace, and load the json from it :

```yml
- attach_workspace:
    at: .
```



```yml
- run:
    name: Merge and check coverage
    command: |
      bundle exec rake coverage:report
```



## View Coverage

If your configuration is correct and tests pass, you can now view the generated test coverage HTML report!



![coverage](https://rubyyagi.s3.amazonaws.com/11-merge-simplecov-circleci/artifact.png)


<script async data-uid="d862c2871b" src="https://rubyyagi.ck.page/d862c2871b/index.js"></script>


