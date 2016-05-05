---
layout: post
title:  "Getting Lazy with Protocols and Dependency Injection"
date:   2016-05-05 08:00:00 -0400
tags: 	swift, testing
---

Welcome back to the exciting world of Dependency Injection! Ok, maybe I get too excited about testing. Last time we talked about [what dependency injection is and how to use it in Swift]({% post_url 2016-04-25-dependency-injection %}). This time we're going to take it one step further with the help of protocols and Swift's `lazy` keyword.

<!--more-->

Going back to our example from last time, we wanted to test the `Musician` class which depends on the `Microphone` class. Here's what our system looks like:

{% highlight Swift %}
class Microphone {
    init() {
        print("Accessing database...")
        print("Observing NSNotifications...")
    }

    func startRecording(favoriteSong: String) {
        // record stuff
    }
}

class Musician {
    var mic = Microphone()
    let favoriteSong: String

    init(favoriteSong: String) {
        self.favoriteSong = favoriteSong
    }

    func sing() {
        self.mic.startRecording(self.favoriteSong)
    }
}

class MusicianTest: XCTestCase {

    func testSingFavoriteSong() {

        // A mock Microphone that that holds on to the last song it recorded
        class MockMicrophone: Microphone {
            var recentSong: String?

            override func startRecording(song: String) {
                self.recentSong = song
            }
        }

        let mockMic = MockMicrophone()
        let prince = Musician(favoriteSong: "Purple Rain")
        prince.mic = mockMic
        prince.sing()

        XCTAssertEqual(mockMic.recentSong, "Purple Rain")
    }
}
{% endhighlight %}

Ok that looks good... no, hold on... look at the init method of the `Microphone` class! What kind of insane person is accessing the database in the init method?! Ugh. It must be the last developer's fault.

If we run our test suite and look at the console, sure enough we'll see "Accessing database..." and "Observing NSNotifications...". Now this really screws up our test plan. Are we going to have to create a database just for this test? And what's going to happen when an NSNotification fires? Our test could act unexpectedly. Welcome to the exciting world of Legacy Code. :(

Fortunately we can rewrite our test code to solve this problem without touching (much) production code. The number one rule of working with Legacy Code is: don't touch it until you can test it!

Our test is having trouble because `MockMicrophone` is a subclass of `Microphone`. We can break this connection by using a **protocol**. Let's give it a try:

{% highlight Swift %}
protocol RecordingDevice {
    func startRecording(favoriteSong: String)
}

class Microphone: RecordingDevice {
    func startRecording(favoriteSong: String) {
        // record stuff
    }
}

class MockMicrophone: RecordingDevice {
    func startRecording(song: String) {
        self.recentSong = song
    }
}

class Musician {
    var mic: RecordingDevice = Microphone()
    // ...
}

{% endhighlight %}

Well, what do we have here? The `Microphone` and `MockMicrophone` now implement the `RecordingDevice` protocol, and the `Musician` class uses a `RecordingDevice` instead of a `Microphone`. Now when we create a MockMicrophone in our test suite, it won't access the database because it doesn't inherit from Microphone. Sweet! Let's run our test suite and take a look at the console...

{% highlight Console %}
Accessing database...
Observing NSNotifications...
{% endhighlight %}

Gah! What happened?! Didn't we get rid of our dependency on the Microphone class? Oh wait, here's the problem:

{% highlight Swift %}
class Musician {
    var mic: RecordingDevice = Microphone()
    // ...
}
{% endhighlight %}

Even though our test function sets the Musician's mic, it does so just a little too late. The mic variable is initialized when the class is initialized and we can see right here that the default value is a new Microphone.

At this point, we should probably just give up and go get a taco. I don't know how to solve this problem and I'm too lazy to...

Wait, that's it! We can use Swift's `lazy` keyword! `lazy` will defer the initialization of a variable until the first time it is used. So let's make the `mic` instance variable lazy and try our test:

{% highlight Swift %}
class Musician {
    lazy var mic: RecordingDevice = Microphone()
    // ...
}
{% endhighlight %}

And now if we look at the console it will look as white as a field of fresh snow. We've got a test suite that runs isolated from `Microphone` and all its dirty, dirty code. And we didn't have to change any production code (all we did was create a protocol which described the functions we already had). A job well done! Now let's go get that taco.

You can find a playground for this article [here](/assets/playgrounds/2016/05/LazyDependencyInjection.playground.zip).