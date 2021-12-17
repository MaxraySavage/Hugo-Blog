---
title: "Spring Boot in the Cloud"
date: 2021-12-15T12:19:03-05:00
author: Maxray
---

# Launch Your Spring Boot Project Fast
A few weeks ago I had a project that I made with Spring Boot that I wanted to show other people.  I spent an inordinately long amount of time wading through strange errors to get it working on Heroku. I don't want you to have to go through what I did. I also don't want to forget what I did and go through it again myself. So I'm writing it down for posterity in the hopes that you won't have to go through the same pain.

## Getting Set Up
First, a note. These steps worked for me, they may not work for you. If something doesn't work, let me know! Click `Suggest Changes` at the top of this article and leave an issue on the Github repo. Thanks!

Second, I set Java 11 and set my packaging setting to War in the spring initializr when I started my project. If you did not do those things, I honestly don't think these steps will work for you. I did try uploading a Jar to Heroku and ended up with some problems that I decided to fix by never doing it again.

Now let's get started.

### System Properties
Heroku needs to find a `system.properties` file to know which version of Java your app uses. 
Make a file in the root directory of your project called `system.properties`.

In the file we only need one line of text: 
```bash
java.runtime.version=11
```
If you are attempting to follow this post with a different version of Java than 11, I'd imagine you should replace 11 with whatever version you are using.

### Double Check your App Works
Pause. Now is a great time to run your app locally and make sure it works. Do that with the Gradle task `bootRun`. 

If it looks good, make *sure* you stop your locally running version of your app (the one I just told you to start with `bootRun`) **BEFORE** moving to the next step.

### Build
We are now going to build the `.war` file that we will upload to heroku.

Sometimes when I have built my app is while it is running locally gradle says the app build was successful but doesn't incorporate any new changes I've made. Hence the warning at the end of the previous step.

So, now that you've made sure your app isn't running locally, run the Gradle task `build`. 

In Intellij, you can find it in the Gradle panel here:
![A screenshot of the Gradle task list with the build task highlighted](/Launch-Your-Spring-Portfolio-Project/gradle-build.png)

Once your project is built you are ready to upload to Heroku!

## Heroku
Now your app is all prepped locally, time to go to the clouds.

### Heroku Account and Tools 

If you don't yet have a Heroku account, head on over to their site and create one now.

You are also going to need the Heroku [command line interface](https://devcenter.heroku.com/articles/heroku-cli) so install that with the instructions appropriate for your operating system.

Lastly, you'll need the Heroku java plugin [^bignote] which can be installed from the command line like so:

```bash
$ heroku plugins:install java
```
Now you're ready to go!

### Actually making the App

First we need to create your app! 

You don't *need* to decide on your own app name because Heroku will make one up for you. But it IS more fun that way!
 
Keep in mind that this app name will show up in your project's URL so make sure not to put spaces or anything that would normally need to be escaped in a URL.

Once you've decided on a name (hereafter referenced as `<app_name>`) you can run the following command on the command line to create your app!

```bash
$ heroku create -n <app_name>
```

The `-n` flag is so Heroku doesn't automatically add a new git remote to your local repository. We won't need it because we are uploading our app manually.

### Upload the War 

Next you'll need to find the `War` file you built earlier within your project folder structure. I was able to find mine in `/build/libs`.

Find your `War` and note its path. 

Now run the following command on the command line.
```bash
$ heroku war:deploy <path_to_war_file> --app <app_name>
```
If your app requires a MySQL connection to start correctly, it won't work yet. 

We need to create the app on Heroku before we can add the database.


### ClearDB
One the app is uploaded, you can add a free tier MySQL database with the following command

```bash
$ heroku addons:add cleardb:ignite -app <app_name>
```

Heroku requires a credit card on file to do this because it *could* cost money. 

However, ignite is the free tier and I have not been charged and I don't think you will either.

Your app should restart on its own, connect to the database and get up and running!

If you have trouble finding your app you should be able to find its url from your Heroku dashboard.

It is that easy! Enjoy!

Also, if you're curious, the project that I did all this work to host is 
* Hosted at: https://hungr-dev.herokuapp.com/
* And the repo is here: https://github.com/MaxraySavage/hungr-lc

## Life isn't about the Application, it's about the Errors you make along the Way
Just because I think it's interesting, here's a list of all the problems I ran into trying to get this to work
* Trying to use Heroku the usual way (by `git push`-ing to a heroku git remote)
    * This was my first attempt and there were so many different issues that I can't even point to one
    * I eventually gave up and decided to try and just upload the `.war` which was so much easier
* Not using a `system.properties` file
    * Everything builds and runs but when you go to the app, you get a `404` error from the Tomcat server
    * Checking the Heroku logs shows no errors
* Using a `.jar` instead of a `.war`
    * When I tried this, ClearDB and my application failed to agree on a TLS version and the handshake failed. I found a lot of people with similar issues and rather than troubleshoot, decided to just stick to deploying `.war` files.

[^bignote]: https://devcenter.heroku.com/articles/war-deployment#deployment-with-the-heroku-cli