---
layout: post
title:  "Swift and Grand Central Dispatch"
date:   2014-06-23
---
I read somewhere the other day (some article on the interweb) someone remark that they "hadn't seen an equivalent of Grand Central Dispatch (GCD) for Swift". It struck me as odd, given GCD works perfectly fine in Swift, in fact I think you end up with a lot nicer syntax. Let me explain.

In Objective-C you use GCD like this
```
dispatch_async(/* a queue /*, ^{
    /* some code to be executed on the queue */
});
```

Note the ugly syntax, the `^` and that dastardly semicolon.

In Swift, *blocks* are replaced by *closures*, and the syntax looks like this.

```
dispatch_async(/* a queue /*, {
    /* some code to be executed on the queue */
})
```

Pretty similar really. However, Swift has a feature called *[trailing closure syntax][trailing-closure]*. This means if a function's last argument is a closure you can omit the parentheses. Now if we define some utility functions for `dispatch_async`ing to the main queue, and to a background queue (which probably covers at least 90% of the usage of GCD) like so

```
func dispatch_to_main_queue(block: dispatch_block_t?) {
    dispatch_async(dispatch_get_main_queue(), block)
}

func dispatch_to_background_queue(block: dispatch_block_t?) {
    let q = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
    dispatch_async(q, block)
}
```

Now we can use trailing closure syntax for much more expressive code, where GCD feels like a first-class language feature rather than a function call.

```
dispatch_to_main_queue {
    /* some code to be executed on the main queue */
}
dispatch_to_background_queue {
    /* some code to be executed on a background queue */
}
```

And thus it was that Swift and Grand Central Dispatch became the best of friends.

[trailing-closure]: https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Closures.html#//apple_ref/doc/uid/TP40014097-CH11-XID_126
 