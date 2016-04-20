---
layout: post
title:  "Ownership, or My First Swift PR Got Merged"
date:   2016-04-19
---

A couple of months ago I was playing with [Soroush Khanlou](http://khanlou.com)'s [composable  light view controllers concept](http://khanlou.com/2016/02/many-controllers/), resulting in [this Swift implementation of his keyboard manager view controller](https://gist.github.com/neilkimmett/932bbf5d8ddc364c2ba6). Note that the gist starts with the lines

```swift
extension UIEdgeInsets {
    static var zero: UIEdgeInsets {
        return UIEdgeInsetsZero
    }
}
```

This is because the Swift standard library ships with `CGRect.zero` for getting an empty `CGRect`, and `CGPoint.zero` for an empty `CGPoint` but _doesn't_ ship with `UIEdgeInsets.zero` 💔. I was discussing this discrepancy with Soroush and he wisely suggested I file a Radar. But then I had a realisation

![Chat with Soroush](/assets/soroush.png)

Thanks to Swift now being open source I could fix the bug myself! This had me infinitely more excited than the arduous process of writing up a Radar. We all know that Radar'ing can be a very arduous, opaque and frustrating process, instead I could fire up my code editor of choice and _actually fix the problem_.

So I found the relevant part of the code, found the relevant part of the unit tests, and [submitted a pull request](https://github.com/apple/swift/pull/1323). [Jordan Rose](https://twitter.com/uint_min) got back to me a few hours later (hours!) to let me know he'd need to consult the frameworks team at Apple as this involved a change to the frameworks overlays. Once UIKit approved the change (which took about a month) my pull request got merged.

What makes this so exciting to me is not just the new, open way of talking directly to Apple enginers but the fact that some of _my_ code is going to be in Apple's frameworks. When new iOS and Mac betas come out at WWDC they'll be running _my_ code, albeit a tiny bit of code. Thats a fundamental change, I feel like I have ownership of the platforms I spent my whole day using. So next time you find a little (or big!) annoyance in Swift don't jump straight for Radar. Maybe you can track down the cause of the issue and fix it yourself 😊.
