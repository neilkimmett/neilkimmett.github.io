---
layout: post
title:  Introducting Street Count
date:   2023-10-22
---

![A screenshot of the promotional website for Street Count](/assets/street-count-site.png)

Today I’m thrilled to announce the release of [Street Count](http://streetcount.app/), a little app for measuring traffic in your neighbourhood. Its designed to be used by local organizers to gather information about how streets are used, and use it to advocate for street safety improvements. The app works by presenting big buttons for you to tap as people (and dogs) stroll through your line of sight, and then offering a convenient way to export the data to a CSV file for easy analysis. The app is designed with privacy in mind: it collects no identifiable information, and the street count data you collect does not leave your device[^export].


### Why


I’m part of a couple of [local groups](http://fortgreeneopenstreets.org/) of [safe streets activists](https://www.clintonhillsafestreets.com), and a couple of years ago we undertook a “street count” in our neighborhood where we counted how many people were walking or cycling or driving along the street. We then use this data to advocate for quality-of-life improvements to our streetscape, such as [pedestrian bulbouts](https://www.nycstreetdesign.info/geometry/curb-extension) and protected bike lanes. We did this first street count in a very lo-fi way: with sheets of paper and clipboards. Our streets get a lot of foot traffic so I found it really challenging to keep up with the tally marks in every column. 

![A photo of the sheet we used to count traffic](/assets/clipboard.jpg)

By the end of my 2 hour shift my hand was causing me a ton of pain[^rsi], and in order to analyze the data I had to manually count everything up. As a professional app developer I saw an opportunity here: I could create an app that just has big easy buttons for someone to hit, and then the app can do all of the counting for you (computers are quite good at counting).

### The technical details


I threw together a quick prototype (SwiftUI makes things like this *so* much easier) and then gave it to some volunteers for the next street count we conducted. To my great delight it worked super smoothly; after a few minutes we found that you don't even need to look down at the phone screen as you build muscle memory for where each categorization (person, cyclist, runner, etc) is. Now that the idea had been validated I set to work (occasionally, over the course of a couple of years, in stolen bits of time on weekends and holidays) turning it into an actual product. I wanted to create both an iOS and Android app, and I’m a huge Kotlin fan (we use it a ton at [ClassPass](https://classpass.com)) so I decided to try out using [Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform.html). This allows you to share your core data models and business logic across platforms, but leaves you to develop your user interface using the native APIs for each platform (in this case SwiftUI and Jetpack Compose for iOS and Android respectively). The business logic for the app is pretty simple (its essentially just...adding up numbers) but I found this a really nice way to work; it – by definition – forces you to separate your business logic from your UI and allows a nice amount of logic sharing without the pitfalls of frameworks that try to abstract over the entirety of the UI APIs of both iOS and Android (and inevitably fall short). I found myself developing an interesting ping-pong sort of development lifecycle, where I'd develop a new feature – shared logic and UI – on one platform, then port it to the other platform, then develop the next feature on that platform, before porting back to the first platform. Which leads me on to a tool that massively accelerated my development: ChatGPT.

I’m a huge fan of [Simon Willison’s framing of ChatGPT as “a pairing assistant”](https://simonwillison.net/2023/Mar/27/ai-enhanced-development/) and I used it a ton in this project. There are a lot of similarities between SwiftUI and Jetpack Compose, and ChatGPT does an absolutely remarkable job when you copypaste in the code for a SwiftUI screen and ask it “convert this to Jetpack Compose” (and vice versa). It obviously only works with simpler examples (which this app is full of) and it doesn’t necessarily spit out fully working code on the first try, but it automates away a lot of the toil of manually converting, and often gets you 50-75% of the way there in a fraction of the time. As an iOS engineer I also used it a lot as a “pairing buddy” to teach me about the Android equivalent of something I already knew in iOS. It also massively helped me with making a website for the app. I didn’t want to ship the app without a site to link to, but also the prospect of writing something from scratch meant I procrastinated for a long time. I’d been meaning to play around with Tailwind CSS for a while, so I just asked ChatGPT “make me a marketing website for a mobile application using Tailwind as the CSS framework” and it wrote a really great first attempt that I then added my own design and content to. Being able to brute force your way through the “blank page problem” is such a powerful tool for creativity.

### Out for the count


I’m so glad the app is in a good enough place that I feel comfortable sharing with the world. I hope that other safe streets groups can use the app in their neighborhood, but regardless I’m proud just to get it out there. In my decade-plus in the tech industry I’ve never shipped my own app (see my [blog post from 2015](/2015/02/11/working-on-side-projects.html) where I aimed to spend more time on side projects 😬), so this is a big personal milestone! I hope to follow it up with more improvements and hopefully some other related projects. Check out the app at [http://streetcount.app/](http://streetcount.app/) , and email me on [info@streetcount.app](mailto://info@streetcount.app) if you have any feedback, I’d love to hear from you!

[^export]: Until you press the export button, at which point the data leaves the app, and enters the app of your choice (email, iCloud, Google Drive etc). The app does not collect any telemetry about what app you choose to export your data to.

[^rsi]: I suffer from pretty bad repetitive strain injury from typing and general computer use. If you do too I strongly recommend reading [this book](https://www.amazon.com/gp/product/0471388432/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1), getting yourself a good ergonomic keyboard, mouse and desk, doing _lots_ of stretches, and seeking professional medical advice.
