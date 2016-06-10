---
layout: post
title:  "for-loops are for suckers: An OO-programmer's guide to map and flatMap"
date:   2016-06-09 22:00:00 -0400
tags: swift
---

For-loops are so 2014. Swift provides us with two new functions that can handle jobs that were traditionally done with a loop. Functional programming languages have been using `map()` and `flatMap()` for years, but they are also extremely useful in object-oriented languages. And Swift handles these functions with speed, safety, and style.

<!--more-->

Let's jump right in with a quick example. Let's say we are building a music app and we want to turn a bunch of dictionaries into Song objects. This pattern should be very familiar to anyone who writes an app that connects to a web service. Let's define a few things first:

{% highlight Swift %}
// A simple struct
struct Song {
    let title: String
}

// Sample data: the first few tracks from a Rolling Stones album,
// represented by an array of dictionaries
let beggarsBanquetData = [
    ["title": "Sympathy for the Devil"],
    ["title": "No Expectations"],
    ["title": "Dear Doctor"]
]

// A function to create a Song based on a dictionary
func dictionaryToSong(dict: [String: String]) -> Song {
    let songTitle = dict["title"] ?? "Unknown"
    return Song(title: songTitle)
}
{% endhighlight %}

## The old way: with a for-loop
And now let's transform the array of dictionaries into an array of Songs:
{% highlight Swift %}
// Create an array of songs from the sample data
var beggarsBanquet0: [Song] = []
for songDict in beggarsBanquetData {
    let song = dictionaryToSong(songDict)
    beggarsBanquet0.append(song)
}
// result: [Song, Song, Song]

{% endhighlight %}

## But can it map?
`map()` is a function that is performed on a collection. It takes one parameter, which is a function called `transform`. It applies the `transform` function to every element in the collection, and then builds a new collection with the new elements. Let's see how we can rewrite the above loop using `map`.

{% highlight Swift %}
let beggarsBanquet1 = beggarsBanquetData.map( { (songDict: [String : String]) -> Song in
    return dictionaryToSong(songDict)
})
// result: [Song, Song, Song]
{% endhighlight %}

Ok that's pretty ugly. Let's quickly make a few stylistic changes. First, we can remove the parameter declaration in the closure and replace the parameter with a $0.

{% highlight Swift %}
let beggarsBanquet2 = beggarsBanquetData.map {
    return dictionaryToSong($0)
}
// result: [Song, Song, Song]
{% endhighlight %}

That looks better, but we can keep going. We don't really need the closure at all. We can just provide the name of the `dictionaryToSong(_)` function. Swift will implicitly pass along the $0 so we don't need that either:

{% highlight Swift %}
let beggarsBanquet3 = beggarsBanquetData.map(dictionaryToSong)
// result: [Song, Song, Song]
{% endhighlight %}

So now the whole thing is one one line. But what have we really accomplished here?

## So map is just a weird-looking for-loop, right?
The first time I saw `map`, I thought "cool story, but that's not as readable as the for-loop was". But after using `map` for a while I have to disagree with that statement. Now I think that it is *more* readable than the for loop and more powerful as well. Here are two reasons why.

### map is Declarative
A for-loop doesn't inherently mean much. It just says *"for each element in a collection, do this."* `map` on the other hand explicitly says *"transform the elements of this collection into something new"*. When you see a map in code, you know exactly what's happening.

![Transformers](/assets/images/2016/06/transformers.gif "Using map() is like saying "transform this collection of cars into a collection of badass robots"")

Using `map` is like saying *"transform this collection of cars into a collection of badass robots"*

### Results are Immutable
Mutability leads to bugs, bugs lead to anger, anger leads to the Dark Side. Fortunately the result of `map` is an immutable collection. Unfortunately the result of a transformation via for loop is mutable. Take a look:

{% highlight Swift %}
let rocksOff = Song(title: "Rocks Off")

// beggarsBanquet0 is the mutable array from our for-loop
beggarsBanquet0.append(rocksOff)
// Bad developer! "Rocks Off" is a song from Exile on Main St! Wrong album!

// beggarsBanquet3 is an immutable array from our map
beggarsBanquet3.append(rocksOff)
// Compile error. Don't even try it!
{% endhighlight %}

## flatMap: It's like map, but without nils
Often we have a transformation function that can produce nil. In our example that could happen if we have a dictionary that cannot be transformed into a Song. Like this:

{% highlight Swift %}
// Sample data: the extended edition of the album with a video at the end
let beggarsBanquetExtData = [
    ["title": "Sympathy for the Devil"],
    ["title": "No Expectations"],
    ["title": "Dear Doctor"],
    ["video": "Video Interview"]
]

// Function to create an optional Song from a dictionary
func dictionaryToOptionalSong(dict: [String: String]) -> Song? {
    guard let songTitle = dict["title"] else {
        return nil
    }

    return Song(title: songTitle)
}

let beggarsBanquet4 = beggarsBanquetExtData.map(dictionaryToOptionalSong)
// result: [Song, Song, Song, nil]
{% endhighlight %}

What can we do in this case? You got it: use `flatMap`.

{% highlight Swift %}
let beggarsBanquet5 = beggarsBanquetExtData.flatMap(dictionaryToOptionalSong)
// result: [Song, Song, Song]
{% endhighlight %}

It's that simple. `flatMap` will throw away any nils rather than add them to the collection. What `flatMap` does behind the scenes is actually really interesting, but that's the subject of a future post. For now all you need to know is that `flatMap` will get rid of nils.

## map and flatMap are Extra Cool
There are several other benefits you get when using `map` and `flatMap`. These are a little bit advanced but they're good to know about.

### Pipelining
Pipelining sounds like a surfing concept but programmers have their own definition of it. It means that you can pass the results of one function to another.

{% highlight Swift %}
func isAboutDoctors(song: Song) -> Bool {
    return song.title.containsString("Doctor")
}

let songsAboutDoctors = beggarsBanquetData.map(dictionaryToSong).filter(isAboutDoctors)
// result: [Song(title: "Dear Doctor")]
{% endhighlight %}

Ride the pipeline! Gnarly, dude!

### Lazy evaluation
This is really awesome, but difficult to show. Check it out:

{% highlight Swift %}
let lazyBeggarsBanquet = beggarsBanquetData.lazy.map(dictionaryToSong)
print(lazyBeggarsBanquet[2])
// "Song(title: "Dear Doctor")"

// lazyBeggarsBanquet[0] and lazyBeggarsBanquet[1] were never initialized
// because we never asked for them!
{% endhighlight %}

Using the `lazy` keyword tells Swift to only perform the `map` function when we actually go to use the values. If you had an enormous collection this could be very performant. Try doing that with your for-loop!

### Refactoring and Parallelization
With `map` we've taken a common action (transforming a collection) and hidden it away behind a function. This is great because we don't care how the function is implemented. And this allows other people (the Swift team, LLVM, or whomever) to refactor the functions and make them faster or safer.

One suggestion we may see in the future is this: since `map` doesn't have to go through the collection sequentially, it would be possible for Swift to hand off pieces of the collection to different processors on a multi-core device, thereby performing `map` in parallel across multiple cores. In that case we could see a 2x, 4x, or more increase in performance!

## A Puzzle
So we've learned that `map` and `flatMap` are awesome. We can use them to transform collections in a safe, fast, and expressive way. But we've really just brushed the surface of what these functions, especially `flatMap`, actually do. I'll cover that in an upcoming post but until then, take a look a this code:

{% highlight Swift %}
let prodigalSon1: Song = Song(title: "Prodigal Son")
prodigalSon1.map(isAboutDoctors)
// Compile error!

let prodigalSon2: Song? = Song(title: "Prodigal Son")
prodigalSon2.map(isAboutDoctors)
// result: false
{% endhighlight %}

What's going on there? [Let me know on twitter](https://twitter.com/DavidNorcott) what you think is happening!

You can find a playground for this article [here](/assets/playgrounds/2016/map_and_flatmap.playground.zip).