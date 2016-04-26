---
layout: post
title:  "Dependency Injection in Swift"
date:   2016-04-25 11:00:00 -0500
tags: 	swift, testing
---

I've recently been working with a large legacy codebase that doesn't have great test coverage (I guess that's redundant since Michael Feathers defines legacy code as [code without tests](https://en.wikipedia.org/wiki/Legacy_code)). As I fix bugs or add features, I also try to add unit tests along the way. But this can get pretty tricky pretty fast and one of the main roadblocks is a class's dependency graph.

<!--more-->

When a class calls out to another class to get data or perform an action, we say it is dependent on the other class. Here's a quick example:

{% highlight Swift %}
class Musician {
    let favoriteSong: String

    init(favoriteSong: String) {
        self.favoriteSong = favoriteSong
    }

    func sing() {
        let mic = Microphone()
        mic.startRecording(self.favoriteSong)
    }
}
{% endhighlight %}

Here we've got a Musician class that can do one thing: sing a favorite song! Now Let's write a test for that.

{% highlight Swift %}
class MusicianTest: XCTestCase {
    func testSingFavoriteSong() {
        let prince = Musician(favoriteSong: "Purple Rain")
        prince.sing()
        // Now what?!
    }
}
{% endhighlight %}

We quickly run into a problem: there's nothing to assert! How do we know if the musician is singing?

The real issue is that the Musician *depends* on the Microphone but our test has no way of accessing the Microphone. We can't check if the Musician is using it correctly (or at all). Another major issue is that we don't know what the Microphone is doing behind the scenes. For all we know it could be starting up a network connection or fetching data from a database. Our dependencies could have dependencies! This is getting awfully complicated.

We need a way to separate out the dependencies in our Musician class. Let's start by pulling the Microphone out into an instance variable.

{% highlight Swift %}
class Musician {
    // the Microphone is now an instance variable
    var mic = Microphone()

    let favoriteSong: String

    init(favoriteSong: String) {
        self.favoriteSong = favoriteSong
    }

    func sing() {
        self.mic.startRecording(self.favoriteSong)
    }
}
{% endhighlight %}

Ok, that's pretty straightforward. Now let's write a test that makes use of this `mic` instance variable. Let's *inject* the Microphone *dependency*.

{% highlight Swift %}
class MusicianTest: XCTestCase {
    func testSingFavoriteSong() {

        // MockMicrophone remembers the last song it recorded.
        class MockMicrophone: Microphone {
            var recentSong: String?

            override func startRecording(song: String) {
                self.recentSong = song
            }
        }

        let mockMic = MockMicrophone()
        let prince = Musician(favoriteSong: "Purple Rain")

        // set the Musician's mic
        prince.mic = mockMic

        prince.sing()

        XCTAssertEqual(mockMic.recentSong, "Purple Rain")
    }
}
{% endhighlight %}

Whoa! Hold on one second! This test just took an interesting turn of events. Did I just declare a new class *inside* of a function?! Coming from an Objective-C world, this blew my mind the first time I saw it. Dear iOS developers: welcome to the 21st century.

But back to the matter at hand. The new class, `MockMicophone` is a subclass of `Microphone` that remembers the last song it recorded. By injecting this new MockMicrophone into our Musician class we now have a way to verify behavior.

In a nutshell, this is what Dependency Injection is. It's just a way to make explicit the dependencies your class has, and allow other objects to override those dependencies.

### DI: Four Ways

There are four ways (that I know of) to accomplish dependency injection. Let's run through them quickly with some examples:

{% highlight Swift %}
// 1. DI via a method parameter
class Musician {
    func sing(withMicrophone mic: Microphone) {
        mic.startRecording(self.favoriteSong)
    }
}

// 2. DI via class initialization
class Musician {
    private var mic: Microphone

    init(mic: Microphone) {
        self.mic = mic
    }

    func sing() {
        self.mic.startRecording(self.favoriteSong)
    }
}

// 3. DI via instance variable with a default value
class Musician {
    var mic = Microphone()

    func sing() {
        self.mic.startRecording(self.favoriteSong)
    }
}

// 4. DI as a service
class Musician {
    func sing() {
        AppContext.sharedContext.mic.startRecording(self.favoriteSong)
    }
}
{% endhighlight %}

The first and second methods are pretty self-explanatory. Many people like these methods because they make the dependencies explicit. When you go to use the Musician class, it is quite obvious what dependencies that class has.

We already explored the third method in this article. This is my preferred method when working with legacy code because it allows you to quickly separate dependencies and write tests without rewriting much of your production code. In the example we saw here, none of my production code that talks to the Musician class needs to change. All I had to do is extract an instance variable and *voilà!* I could start testing.

The fourth is pretty interesting and powerful but I'm not sure I'd recommend it. It removes the dependency on Microphone but it adds a dependency on new singleton called AppContext. What makes this powerful is that your test suite and your production code could have different AppContexts, allowing you to make sweeping changes quickly.

### Conclusion

So there you have it! I once heard dependency injection called "a ten-dollar word for a ten-cent concept". It's a simple yet powerful tool to break up dependencies in your code and increase testability.
