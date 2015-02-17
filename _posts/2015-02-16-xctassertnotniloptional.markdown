---
layout: post
title:  XCTAssertNotNilOptional
date:   2015-02-16
---

Swift is great, but all of the Cocoa frameworks are written in Objective-C. This sometimes results in a tension between Swift and the frameworks. Today we'll talk about a couple of helpers we can add to XCTest to help ease this tension.

Lets say we have a struct in our model layer like this

{% highlight swift %}
struct Thing {
    let name: String
    init?(json: [String: String]) {
        if let name = json["name"] {
            self.name = name
        }
        else {
            return nil
        }
    }
}
{% endhighlight %}

Now, lets say we want to write a unit test checking that this [failable initializer](https://developer.apple.com/swift/blog/?id=17) returns us a `Thing` when given valid JSON

{% highlight swift %}
func test_CreatesThing_WhenGivenJSONWithName() {
    let thing = Thing(json: ["name": "The Thing"])
    XCTAssertNotNil(thing, "Expected to create thing")
}
{% endhighlight %}

Unfortunately this gives us a nasty compiler error. In Swift 1.1 we get…

!["'Thing' is not identical to 'AnyObject'"](/assets/xctest_swift1.1.png)

…and in Swift 1.2 we get…

!["Cannot invoke 'XCTAssertNotNil' with an argument list of type (Thing?, String)"](/assets/xctest_swift1.2.png)

This is because our struct doesn't conform to the `AnyObject` protocol. What we'd like to do is create our own generic helper that can take _any_ optional value, and assert whether it is nil or not. A first pass looks a little something like this

{% highlight swift %}
func XCTAssertNotNilOptional<T>(expression: @autoclosure () -> T?, message: String) {
    let optional = expression()
    let isNonNil = optional != nil
    XCTAssertTrue(isNonNil, message)
}
{% endhighlight %}

In Swift 1.2 the way `@autoclosure` keyword works has changed so the function prototype needs to be…
{% highlight swift %}
func XCTAssertNotNilOptional<T>(@autoclosure expression:  () -> T?, message: String)
{% endhighlight %}

This little helper looks great, until we run our tests, and discover that our assertion fails at the wrong line in our code. Xcode points out the failure _within_ our helper, rather than at the callsite.

![Assertion failure is displayed by Xcode on the wrong line of our code](/assets/xctest_wrongline.png)

This is suboptimal. However if we take a look at the prototype of `XCTAssertTrue` we find a couple of things that look like they might be helpful

{% highlight swift %}
func XCTAssertTrue(expression: @autoclosure () -> BooleanType, _ message: String = default, file: String = default, line: UInt = default)
{% endhighlight %}

Take a look at those `file` and `line` parameters with default values. Maybe we could pass in our own values for those parameters to tell the test runner where the assertion failure originates from? [Apple's Swift blog](https://developer.apple.com/swift/blog/?id=15) comes to our rescue:

> Swift has a number of built-in identifiers, including `__FILE__`, `__LINE__`, `__COLUMN__`, and `__FUNCTION__`. These expressions can be used anywhere and are expanded by the parser to string or integer literals that correspond to the current location in the source code. 

> …

> These identifiers expand to the location of the caller when evaluated in a default argument list

This means if we add `file` and `line` parameters to our helper, with default values of `__FILE__` and `__LINE__` respectively, like so…

{% highlight swift %}
func XCTAssertNotNilOptional<T>(expression: @autoclosure () -> T?, message: String, file: String = __FILE__, line: UInt = __LINE__) {
    let optional = expression()
    let isNonNil = optional != nil
    XCTAssertTrue(isNonNil, message, file: file, line: line)
}
{% endhighlight %}

…then our assertion failure happens at the correct point in our code

![Assertion failure is displayed by Xcode on the correct line of our code](/assets/xctest_rightline.png)

Huzzah! We can define a similar helper for checking if an optional is nil. For completeness here are the two helpers in Swift 1.1 and Swift 1.2

{% highlight swift %}
func XCTAssertNotNilOptional<T>(expression: @autoclosure () -> T?, message: String, file: String = __FILE__, line: UInt = __LINE__) {
    let optional = expression()
    let isNonNil = optional != nil
    XCTAssertTrue(isNonNil, message, file: file, line: line)
}

func XCTAssertNilOptional<T>(expression: @autoclosure () -> T?, message: String, file: String = __FILE__, line: UInt = __LINE__) {
    let optional = expression()
    let isNil = optional == nil
    XCTAssertTrue(isNil, message, file: file, line: line)
}
{% endhighlight %}
<figcaption>Swift 1.1</figcaption>

{% highlight swift %}
func XCTAssertNotNilOptional<T>(@autoclosure expression:  () -> T?, message: String, file: String = __FILE__, line: UInt = __LINE__) {
    let optional = expression()
    let isNonNil = optional != nil
    XCTAssertTrue(isNonNil, message, file: file, line: line)
}

func XCTAssertNilOptional<T>(@autoclosure expression:  () -> T?, message: String, file: String = __FILE__, line: UInt = __LINE__) {
    let optional = expression()
    let isNil = optional == nil
    XCTAssertTrue(isNil, message, file: file, line: line)
}
{% endhighlight %}
<figcaption>Swift 1.2</figcaption>

Hopefully this has helped you understand how to write your own XCTest helpers, and understand the role of identifiers like `__LINE__` and `__FILE__`.