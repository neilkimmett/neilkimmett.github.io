---
layout: post
title:  "Swift and Grand Central Dispatch"
date:   2014-06-23
---
I read somewhere the other day (some article on the interweb) someone remark that they "hadn't seen an equivalent of Grand Central Dispatch (GCD) for Swift". It struck me as odd, given GCD works perfectly fine in Swift, in fact I think you end up with a lot nicer syntax. Let me explain.

In Objective-C you use GCD like this
{% highlight objective-c %}
dispatch_async(/* a queue /*, ^{
    /* some code to be executed on the queue */
});
{% endhighlight %}

Note the ugly syntax, the `^` and that dastardly semicolon.

In Swift, *blocks* are replaced by *closures*, and the syntax looks like this.

{% highlight swift %}
dispatch_async(/* a queue /*, {
    /* some code to be executed on the queue */
})
{% endhighlight %}

Pretty similar really. However, Swift has a feature called *[trailing closure syntax][trailing-closure]*. This means if a function's last argument is a closure you can omit the parentheses. Now if we define some utility functions for `dispatch_async`ing to the main queue, and to a background queue (which probably covers at least 90% of the usage of GCD) like so

{% highlight swift %}
func dispatch_to_main_queue(block: dispatch_block_t?) {
    dispatch_async(dispatch_get_main_queue(), block)
}

func dispatch_to_background_queue(block: dispatch_block_t?) {
    let q = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
    dispatch_async(q, block)
}
{% endhighlight %}

Now we can use trailing closure syntax for much more expressive code, where GCD feels like a first-class language feature rather than a function call.

{% highlight swift %}
dispatch_to_main_queue {
    /* some code to be executed on the main queue */
}
dispatch_to_background_queue {
    /* some code to be executed on a background queue */
}
{% endhighlight %}

And thus it was that Swift and Grand Central Dispatch became the best of friends.

**Update**: [Tom Adrianssen](http://www.twitter.com/Inferis) has crafted an even nicer API to GCD, [check out the gist on Github](https://gist.github.com/Inferis/0813bf742742774d55fa). Using nested classes you can start doing cool stuff like `dispatch.async.bg { /* thing */ }` or `dispatch.sync.main { /* thing */ }`

[trailing-closure]: https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Closures.html#//apple_ref/doc/uid/TP40014097-CH11-XID_126
 