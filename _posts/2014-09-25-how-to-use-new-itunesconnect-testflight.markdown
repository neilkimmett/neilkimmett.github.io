---
layout: post
title:  How to use the new iTunes Connect TestFlight for internal testing
date:   2014-09-25
---

Back in February this year [Apple bought Burstly](http://recode.net/2014/02/21/apple-confirms-burstly-buy/), the owner of the TestFlight beta distribution platform. Then at WWDC in June Apple announced the [newly revamped TestFlight service](https://developer.apple.com/app-store/Testflight/) which is integrated into [iTunes Connect](http://itunesconnect.apple.com). [I’ve written about using TestFlight before](http://itunesconnect.apple.com), this article will be an introduction to the hot new way to use TestFlight.

First things first, you’re going to need to add an app to iTunes Connect. Click on _My Apps_, then hit the + button. Fill in the presented form and view before your very eyes a new app!

![Adding a new app in iTunes Connect](/assets/itunes_testflight_add.png)
<figcaption>Adding a new app in iTunes Connect.</figcaption>

Next you’ll need some people to test your app. Head over to the _Users and Roles_ section, hit the plus button, fill in your tester’s details and on the next screen assign them the _Technical_ role (or _Admin_ if you’re feeling dangerous I guess). On the final screen (not shown) don’t bother checking any notifications, unless you want your tester to get a bunch of emails they probably don’t care about.

![Adding a new tester in iTunes Connect](/assets/itunes_testflight_user.png)
<figcaption>Adding a new tester in iTunes Connect.</figcaption>

Note that [you can only have one iTunes Connect account per email address](https://twitter.com/drbarnard/status/514079953084641281), so the tester may need to sign up for a new account using a different email address if they’re already a member of a different iTunes Connect team. I also noticed that if you invite someone using an email thats already associated with an Apple ID you can change the name on their account when you invite them. Don’t use it for evil though. Definitely don’t do that.

Right, now you’ll need to wait for the tester to accept your invitation. Once they’ve done so go to _Users & Roles_ and click on their freshly added account. Then you’ll want to turn on the _Internal Tester_ switch over on the right hand side.

![Turning a user into a tester](/assets/itunes_testflight_tester_switch.png)
<figcaption>Turning a user into a tester.</figcaption>

Finally, you’ll need to add them as a tester to your app. Head to _My Apps > Your App > Prerelease > Internal Testers_, check the checkbox by their name then hit the Save button.

![Turning a user into a tester](/assets/itunes_testflight_add_to_app.png)
<figcaption>Adding your tester to your app.</figcaption>

Now next time you upload a build using Xcode (the usual _Product > Archive_ followed by hitting _Submit_ on the generated archive) your tester will get an email inviting them to start testing the app. Bingo.

Note that this is for internal testers. You can invite up to 25 members of your team to test an app, and they can each test the app on up to 10 devices. As of writing the new external testing TestFlight isn’t opened up yet. When it is (I’m assuming some time after Apple has recovered from launching the iPhones 6, iOS 8 and soon OS X Yosemite) you’ll be able to invite up to 1,000 testers using just their email address. Unfortunately the app will need to go through a Beta App Review, similar to the regular App Review for submitting to the App Store (but presumably less stringent/exhaustive?). One final note: when you upload your build to iTunes Connect it goes through the standard automated private API check. This is kinda annoying if you want to include some debugging/testing tools that rely on private API without obfuscating the use of said private APIs. Oh well, there’s always [HockeyApp](http://hockeyapp.net) :)


