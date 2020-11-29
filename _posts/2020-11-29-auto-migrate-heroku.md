---
layout: post
title: Automatically migrate database when pushing to Heroku
---

Heroku makes deploying Ruby web app a breeze, however it doesn't run `rake db:migrate` automatically each time you push a new commit to it.



You might have seen the error screen below after deploying your Rails app on Heroku:

![error](https://rubyyagi.s3.amazonaws.com/15-auto-migrate-heroku/wrong.png)



I used to freak out when seeing this, but most of the time it is caused by me forgetting to run `rake db:migrate` after deploying a feature which has new database migrations.



If you view logs of the app, and see there's an error saying 'relation "table_name" does not exist' , it means the database migration is not run hence no table is created.

![log saying table name does not exist](https://rubyyagi.s3.amazonaws.com/15-auto-migrate-heroku/check_log.png)



Wouldn't it be good if there's a way to tell Heroku to automatically run "rake db:migrate" after each git push? 🤔



## Heroku release phase

Luckily Heroku has a feature named '[release phase](https://devcenter.heroku.com/articles/release-phase)', it allow us to run tasks after a git push is received, but before the app is deployed.

![release phase](https://rubyyagi.s3.amazonaws.com/15-auto-migrate-heroku/release-phase.png)



From [Heroku documentation](https://devcenter.heroku.com/articles/release-phase) : 

> Release phase enables you to run certain tasks before a new [release](https://devcenter.heroku.com/articles/releases) of your app is deployed. Release phase can be useful for tasks such as:
> 
> - Sending CSS, JS, and other assets from your app’s slug to a CDN or S3 bucket
> - Priming or invalidating cache stores
> - Running database schema migrations





We can specify the commands to run during release phase in the **Procfile** file, using the `release` process type :

```yml
release: rake db:migrate
web: bundle exec puma -t 5:5 -p ${PORT:-3000} -e ${RACK_ENV:-development}
```

In the above **Procfile** , the "rake db:migrate" command will be run during release phase.



If you want to run multiple commands in the release phase, you can put the commands into a single bash file (eg: release-tasks.sh, located at the root of your repository), then execute the bash file in the release phase like this : 

```yml
release: ./release-tasks.sh
web: bundle exec puma -t 5:5 -p ${PORT:-3000} -e ${RACK_ENV:-development}
```


With this, you won't need to run migration task manually after each git push, and this can make continuous deployment simpler.

<script async data-uid="d862c2871b" src="https://rubyyagi.ck.page/d862c2871b/index.js"></script>