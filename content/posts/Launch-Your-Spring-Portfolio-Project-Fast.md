---
title: "Spring Boot in the Cloud"
date: 2021-12-15T12:19:03-05:00
author: Maxray
---

# Launch Your Spring Boot Project Fast
A few weeks ago I had a project that I made with Spring Boot that I wanted to show other people.  I spent an inordinately long amount of time wading through strange errors to get it working on Heroku. I don't want you to have to go through what I did. I also don't want to forget what I did and go through it again myself. So I'm writing it down for posterity in the hopes that this will be faster for you!

## Getting Set Up
First, a note. These steps worked for me, they may not work for you. If something doesn't work, let me know! Click `Suggest Changes` at the top of this article and leave an issue on the Guthub repo. Thanks!

Second, I set Java 11 and set my packaging setting to War in the spring initializr when I started my project. If you did not do those things, I honestly don't think these steps will work for you. I did try uploading a Jar to Heroku and ended up with some very strange problems that I decided to fix by never doing it again.

Now let's get started.

### System Properties
Heroku needs to find a `system.properties` file to know which version of Java your app uses. 
Make a file in the root directory of your project called `system.properties`.

In the file we only need one line of text: 
```bash
java.runtime.version=11
```
If you are attempting to follow this post with a different version of Java than 11, I'd imagine you should replace 11 with whatever version you are using.

### Build
Pause. Now is a great time to run your app locally and make sure it works. Do that with the Gradle task `bootRun`. If it looks good, run the Gradle task `build`. This will make a `war` file that we can upload to Heroku.

![A screenshot of the Gradle task list with the build task highlighted](/Launch-Your-Spring-Portfolio-Project/gradle-build.png)

Once your project is built you are ready to upload to Heroku!

## Heroku

If you don't yet have a Heroku account, head on over to their site and create one now.

You are also going to need the Heroku [command line interface](https://devcenter.heroku.com/articles/heroku-cli) so install that with the instructions appropriate for your operating system.

Lastly, you'll need the Heroku java plugin [^bignote] which can be installed from the command line like so:

```bash
$ heroku plugins:install java
```
Now you're ready to go!
You'll need to find the `War` file you built earlier within your project folder structure. I was able to find mine in `/build/libs`.

Find your `War` and note its path. 

Now run the following command on the command line.
```bash
$ heroku war:deploy <path_to_war_file> --app <app_name>
```
If your app requires a MySQL connection to start correctly, it won't work yet but we needed to create the app on Heroku before we can add the database.

One the app is uploaded, you can add a free tier MySQL database with the following command

```bash
$ heroku addons:add cleardb:ignite -app <app_name>
```

Heroku requires a credit card on file to do this but even so, `ignite` is free and I haven't been charged as of yet.

Your app should restart on its own, connect to the database and get up and running!

If you have trouble finding your app you should be able to find its url from your Heroku dashboard.

It is that easy! Enjoy!

[^bignote]: https://devcenter.heroku.com/articles/war-deployment#deployment-with-the-heroku-cli