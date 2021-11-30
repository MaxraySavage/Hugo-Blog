---
title: "Launch Your Spring Portfolio Project Fast"
date: 2021-11-30T12:19:03-05:00
---

# Launch Your Spring Code Fast - 101
You are building a portfolio project using Gradle, Java 11, Hibernate, JPA, MySQL and Srping Boot and you want to launch a cloud instance of your project so people can check it out and tell you how cool it is. I too have had this desire! And I did it! And so can you! With Heroku!

### Database Config
Heroku needs a `system.properties` file to know which version of java your app uses. 
Make a file in the root directory of your project called `system.properties`.

In the file we only need one line: 
```bash
java.runtime.version=11
```
Update with whatever version of Java you are using.

Now is a great time to run your app locally and make sure it will build and run. Do that with the Gradle task `bootRun`. If all looks good, go to `Gradle` and run the task `build`. This will make a `war` file that we can upload to Heroku.

![A screenshot of the Gradle task list with the build task highlighted](/Launch-Your-Spring-Portfolio-Project/gradle-build.png)

Once your project is built you are ready to upload to Heroku!

### Heroku

If you don't yet have a Heroku account, head on over to their site and create one now.

Also we are going to need the Heroku [command line interface](https://devcenter.heroku.com/articles/heroku-cli) so install that with the instructions appropriate for your operating system.

Lastly, we'll need the Heroku java plugin [^bignote] which can be installed like so:

```bash
$ heroku plugins:install java
```
Now we're ready to go!
You'll need to find your application's `.war` within your project folder structure. I was able to find mine in the `./build/libs` directory.

Find it and note its path. 

Now run the following command using whatever you want as your app name
```
$ heroku war:deploy <path_to_war_file> --app <app_name>
```
Your app won't work immediately but we need to create the app on heroku before we can add the database.

One the app is uploaded, we can add a free tier MySQL database with the following command

`heroku addons:add cleardb:ignite -app your_app_name`

I believe Heroku requires a credit card on file to do this but `ignite` is the free tier so hopefully you will survive financially.

Your app should restart on its own and connect to the database and get up and running!

If you have trouble finding your app you should be able to find its url from the heroku dashboard.

It is that easy!

[^bignote]: https://devcenter.heroku.com/articles/war-deployment#deployment-with-the-heroku-cli