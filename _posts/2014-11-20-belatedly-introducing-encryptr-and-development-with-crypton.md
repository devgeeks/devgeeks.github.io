---
layout: post
title: "(Belatedly) Introducing: Encryptr (and Development With Crypton)"
description: ""
category: 
tags: [cordova, phonegap, encryptr, spideroak]
---
{% include JB/setup %}


**“A free, open source password manager and e-wallet. Zero-Knowledge. Cloud-based. Private.”**

If you would like to see it in action, my colleague David Dahl has made a [video of it on Linux](https://monocleglobe.wordpress.com/2014/11/19/encryptr-zero-knowledge-essential-information-storage/).

#### First, An Origin Story

In December 2013, I took two weeks off (for certain values of “off”) to travel with my wife and children up to Brisbane to have Christmas with my parents. I used that break (among other things) to spend some time away from my day to day dev tasks by exploring the new [Crypton](https://crypton.io/) privacy framework being built at [SpiderOak](http://spideroak.com). I decided the best way to really learn it was to actually write a small example app. When I had been talking about Crypton at meetups, I had thought that a simple password manager would be a great use-case for the framework. <!--break-->

TL;DR: Using Crypton, I was able to write the greater part of a privacy-focussed, zero-knowledge password manager in two weeks… while also spending time with my family.

Encryptr was the result. It is open source, and [available on GitHub](https://github.com/devgeeks/Encryptr).

Admittedly, the TL;DR above glosses over quite a bit. Between other priorities, waiting for Crypton to reach a point where it had at least been externally audited, refactoring the Backbone integration, and having to learn how to package and sign desktop apps, I did not release Encryptr as an actual “product” for another eight months. But those are topics for other posts.

#### Authentication

One of the “hard parts” of app development is authentication. With all of the breaches lately, no one wants to be storing passwords anymore, much less storing the master password for a password manager. Crypton solves that by using the zero-knowledge [“Secure Remote Password”](http://srp.stanford.edu/) protocol (SRP).

Registering a new user account is very straightforward:

```javascript
crypton.generateAccount('username', 'passphrase', function(error, account) {
  if (error) {
    // Handle the error
  } else {
    // Use the account
  }
});
```

Then, authenticating an existing user is as simple as:

```javascript
crypton.authorize('username', 'passphrase', function(error, session) {
  if (error) {
    // Handle the error
  } else {
    // Use the session
  }
});
```

It was amazing how freeing that was. I didn’t have to decide how I was going to handle auth tokens, or worry if I was storing the user’s password with sufficient encryption. Just [crypton.authorize()](https://crypton.io/docs/concepts/sessions.html), and on to the next part of the app.

#### Backbone integration

Crypton stores data objects in [“Containers”](https://crypton.io/docs/concepts/containers.html). Containers are named and contain an object called “keys” (not crypto keys… keys in the key/value sense) where the actual data is stored. Encryptr is a Backbone app, and although my first integration between Crypton and Backbone was not to my liking, the actual mapping was not that hard. In my first attempt I made some architectural decisions that turned out to be sub-optimal. Originally I had a single Container for the entire app. This original Container was mapped to a single Collection in Backbone, with each Model stored in the Container’s keys object. Although this kept the mapping between Backbone and Crypton very very clean, it meant I would be missing out on some neat features of Crypton like Container sharing. If the user’s entire list was one Container like it was originally, it would be impossible to share an individual password entry with someone else. It would have been all (of their entries) or nothing. It needed to be refactored.

The refactor changed it so that there is a single Container that acts as an index of the password entries (and yes, still maps to a Backbone Collection), but then that index points at the individual entries that are each in their own container. These Containers are mapped to Models. It’s not perfect. Not everything available in Backbone is mapped to Crypton, but so far [it has worked quite well](https://github.com/devgeeks/Encryptr/blob/master/www/js/backbone.crypton.js). I may not have added password sharing to Encryptr yet, but at least now it’s possible.

At it’s heart is Crypton’s session.load() that loads the container with a name that matches the model’s id, then returns its keys:

```javascript
session.load(model.id, function(err, container) {
  if (err) {
    return errorHandler(err, options);
  }
  return successHandler(container.keys);
});
```

This Backbone adapter means that I no longer even really think in terms of Crypton, now I am just writing a Backbone app and everything “just works”.

Cordova for Mobile, and Node-Webkit for the Desktop

Encryptr is currently available for Android, Mac OS X, Linux and Windows. As I am primarily a mobile developer, the app started as just another Cordova/PhoneGap app. However, a useful cloud-backed password manager would also have to work on a user’s desktop. This is where Node-Webkit came in.

Once I had decided on [node-webkit](https://github.com/nwjs/nw.js) for the desktop versions, the bulk of the work was done by the [grunt-node-webkit-builder](https://github.com/nwjs/grunt-nw-builder) Grunt plugin. Aside from [configuring the plugin](https://github.com/devgeeks/Encryptr/blob/master/Gruntfile.js#L160-L172), it mostly just required adding [another package.json file](https://github.com/devgeeks/Encryptr/blob/master/www/package.json) to the app’s www folder.

Obviously, there is a bit more to it before it’s ready to actually release, such as fixing the icon on windows, creating the various installers and signing them. But again, those are really stories in themselves (worthy of their own posts). The node-webkit wiki has many helpful resources about some of those extra steps. I was just pleased that I was able to add desktop platforms outside of Cordova from the same codebase with very few changes.

#### The Bad News: iOS

This is where crypto with JavaScript falls down a bit. It eats CPU power for breakfast. Many Cordova/PhoneGap developers think they need the new WKWebView in iOS 8 so they can have access to the JIT and have faster JS performance. The WKWebView certainly does boost performance of JavaScript. It is about four times faster than the JIT-less UIWebView normally used by hybrid apps. The actual truth, however, is that most apps (games excepted) are not bottlenecked by the speed of their JavaScript. There are many other factors that play a much greater role in the speed of Cordova/PhoneGap apps.

Unfortunately, speed is the greatest bottleneck in an app that performs crypto with JavaScript in a mobile device. As an example, Encryptr takes almost two whole minutes just to log in on my iPhone 5, much less display any entries. This means that until the [WKWebView is available to Cordova apps](https://shazronatadobe.wordpress.com/2014/09/18/cordova-ios-and-ios-8/), there will be no version for iOS. My only option is to either wait for Apple to finally [fix that bug](http://www.openradar.me/radar?id=5839348817723392) and release it in a version of iOS 8, wait for the Cordova team to [come up with a workaround](https://issues.apache.org/jira/browse/CB-7539), or to implement the crypto in Onjective-C (ick).

#### The Future

Hopefully, the near future will bring an iOS version in some shape or form. Until then, I have a great many other feature requests both obvious and surprising. Some I knew before I even launched, and some I hadn’t even considered. That’s the joy of an MVP. Get it out, get it being used, and get feedback on what it’s missing.

Here are some of the features coming in the next few releases:

* **Changing your master passphrase**. This is a Crypton limitation that is currently being resolved
* **Obscuring the display of passwords**. In the app to prevent them being seen “over your shoulder”, etc
* **Import / export of entries**. This is important for users switching to or from Encryptr
* **Offline access**. At the moment, it’s cloud only. I would like it to be offline by default and just use the cloud to sync
* **Internationalisation**. Almost 45% of the downloads from the Android Play store alone are from devices with a primary language other than English
* **Sharing of entries**. Securely share passwords or other data with your wife, business partner, etc
* **And many more**. Organisation of entries by ‘tags’, a full password generator, filtering entries, and whatever else I can add without messing up the simplicity that is one of my favourite features of the app
