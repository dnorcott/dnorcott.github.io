---
layout: post
title:  "What's the big deal with Swift?"
date:   2016-03-18 15:00:00 -0500
categories: swift
---

![Paul McCartney](/assets/images/2016/03/Paul-McCartney.jpg "Paul McCartney, snazzy dresser. Probably not a Swift programmer.")

Paul McCartney, snazzy dresser. Probably not a Swift programmer.

I get it. I really do. I've been writing Objective-C since iOS 3 dropped in 2009. At first I thought it was just some crazy language cooked up by equally crazy engineers at NeXT, but then I began to see the beauty in all those square brackets.

<!--more-->

When Swift was announced I was equal parts excited to see the Next Big Thing™ and sad to hear that my beloved Objective-C would no longer be the King of iOS. For over a year I watched Swift from the sidelines, waiting for the dust to settle, waiting to see what this new language would grow up to be. The more I saw, the less I liked. `var` reminded me of \**shudder\** JavaScript. All those question marks made the language seem unsure of itself. And what was up with `.map()`?

Objective-C is great. It's battle-tested. It's object-oriented, memory-efficient, there is still *so much* for me to learn about it, and Apple has promised that it will still be a first-class citizen in their ecosystem. So why should I bother to learn Swift?

As I write this, Paul McCartney is on the radio, [singing][live_and_let_die] \[1\]:

> What does it matter to you? If you've got a job to do you gotta do it well.

Good point, Paul. Let's stick with Objective-C and get the job done.

But here's the thing: one weekend I decided to fire up Xcode, press `New Project...` and choose `Language: Swift`. The rest of the day was spent with the [Swift Programming Language Guide][swift_book], fighting with the compiler, and enduring a crisis of meaning (which is actually just an average Saturday for me). But I emerged in awe of Swift and its power.

Swift optionals will eliminate an entire class of bugs from our apps. I hope I don't need to convince you how great generics are. They're now available in Objective-C, but they're still nicer in Swift. Look at treatment given to structs, tuples, lazy variables, constants, and closures (which are actually readable!). And have you seen [what you can do with the lowly enum][swift_enums]?!

And those are just language *features*. Swift allows you to go one step further and change the way that you think about a problem. If you write object-oriented code, you'll find plenty to love. If you write protocol-oriented-functional-reactive code (or whatever the kids are doing these days) you can do that too.

I'm not going to convince anyone to write it. Eventually the massive inertia of Swift will sweep us all along. But I hope in upcoming posts I can convince you to like it.

But back to Sir Paul. In the very next line he lays down the truth:

> You used to say "live and let live". But if this ever-changing world in which we live in makes you give in and cry, say "live and let die".

The King is dead. Long live the King.

\[1\] Yeah I know that's the Guns N' Roses version, but I couldn't find an official Paul McCartney version. Plus GnR rules.

[live_and_let_die]: https://www.youtube.com/watch?v=6D9vAItORgE
[swift_book]: https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID309
[swift_enums]: https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html

Image Source: [https://en.wikipedia.org/wiki/File:Paul-mccartney-1350317931.jpg](https://en.wikipedia.org/wiki/File:Paul-mccartney-1350317931.jpg)