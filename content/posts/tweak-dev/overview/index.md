+++
title = 'Tweak Dev - Overview'
date = 2025-01-22
weight = 1
summary = "The tools and concepts we will be using" 
+++

<!-- {{< youtube dQw4w9WgXcQ >}} -->
<!---->
<!-- This post is available in video-format too -->

In this series, I'll teach you all you need to know to get started with iOS jailbreak tweak development.
This blog will act as an overview, explaining the different tools and concepts we'll use to make tweaks.

## Choosing a language

Firstly we need to decide which language to use,
technically you could use any language which has access to the objc runtime
but the two main languages used are objc and swift.

### Objective-C

Objective-C is a superset of c, meaning all c code is valid objc code. Objc adds object orientated programming concepts into c.
It's used very commonly in Apple's ecosystem.

With the build system we'll be using, objc-based projects have a very simple structure.
Unlike in swift we don't have to worry about interoperability with other languages.

#### Logos

We can make use of a preprocessor, [logos](https://theos.dev/docs/logos),
which abstracts a lot of the runtime logic we'd otherwise have to deal with.
However, since it's a preprocessor it means we won't be able to take advantage of the clangd LSP
and so we won't have access autocomplete, inline documentation and other quality of life features

#### Substrate

The alternative is interacting with [substrate](https://www.cydiasubstrate.com/) directly.
However it is quite verbose, which sometimes can be great
but 99% of the time it just adds overhead we don't need to worry about.

### Swift

In my opinion swift has a much nicer syntax which, for me, makes it easier to write in.
It's also very similar to the languages I already knew when learning it which I believe made it easier to pick up

#### Orion DSL

Like how objc has logos swift has Orion,
unlike logos, orion isn't a preprocessor and instead a DSL which means we have access to the LSP still.
So we can make use of autocomplete, inline documentation, goto-definition, a formatter etc

#### New abi pains

Apple's new arm64e abi (as of writing) is proprietary meaning you can only build for it from a mac,
which makes it difficult for people like myself who don't own a mac
the new abi is used in the majority of system processes on arm64e devices so we'll need to deal with it for writing most of our tweaks
There is a package on most jailbreaks called `oldabi` however, this only works for objc not swift code.
The solution I and a few others have come up with is using a mac-vm with some shell scripts to make interacting with it easier.
I'll cover setting up this vm and tooling the next post

### It's up to you

It's entirely up to you which language you choose, for the reasons mentioned above, I'll be using swift in this series
but most will apply to objc tweaks too so you can still follow along

## Hooks

Now for the fun bit

Hooking is the name used for swizzling in tweak dev.
Swizzling is a niche objc-runtime feature which allows us to swap the implementation of a pre-existing method with our own
Which in turn allows us to modify the behaviour of apps and with a jailbreak, iOS itself.

Say we have `SomeClass`

```swift
class SomeClass {}
```

And that class has `someMethod` which returns a string saying "Original Return"

```swift
class SomeClass {
    func someMethod() -> String {
        return "Original Return"
    }
}
```

And `someOtherMethod` prints it's return value

```swift
class SomeClass {
    func someMethod() -> String {
        return "Original Return"
    }

    func someOtherMethod() {
        print(someMethod()) // Prints "Original Return"
    }
}
```

Now in our tweak, we can create a class, named whatever makes sense to us

```swift
class SomeClassHook {}
```

This class will extend the `ClassHook` protocol, provided by Orion

```swift
class SomeClassHook: ClassHook<> {}
```

Inside the generics we'll put the class we wish to hook, in our case that's `SomeClass`

```swift
class SomeClassHook: ClassHook<SomeClass> {}
```

We can then copy the signature of `someMethod` from `SomeClass` which we defined earlier

```swift
class SomeClassHook: ClassHook<SomeClass> {
    func someMethod() -> String {
        return "Original Return"
    }
}
```

But we'll change the return value

```swift
class SomeClassHook: ClassHook<SomeClass> {
    func someMethod() -> String {
        return "Hooked Return 🔥"
    }
}
```

Now even though we haven't done anything to it, `someOtherMethod` will now print our modified string.


```swift
class SomeClass {
    // ...

    func someOtherMethod() {
        print(someMethod()) // Prints "Hooked Return 🔥"
    }
}
```

As I'll show in later posts it's also possible to call the original implementation of a hooked method from our hook

This is extremely flexible as it allows us to run our code before the original method, after it, or if we wanted both before and after

## Theos

Although it's not strictly required to make a tweak it greatly simplifies it as it automates the task of building and packaging our tweaks.

It's cross platform so it's fully functional, apart from the new abi issues I mentioned earlier, on all platforms.

It's highly configurable with Makefiles allowing us to fully customise how our tweak is built and packaged

The documentation for theos is available at [theos.dev](https://theos.dev)
And their discord server, which is a great place to get help with tweak dev, is available at [theos.dev/discord](https://theos.dev/discord)

## Reverse Engineering

Reverse engineering is like trying to find the recipe to a cake with only the final product, tricky right?
Well there's a few tools we can use to help us.

### Dynamic

There's dynamic tooling which attaches itself to the running process allowing us to see whats going on live,
the main one we'll be using is FLEX, which allows us to simply tap on what we want and see that view's properties and methods on-device.

You've also likely used another dynamic tool before, a debugger like `gdb` is also able to be used a dynamic reverse engineering tool too!

### Static (Decompiler)

There's also static tools, decompilers, which will take the assembly machine code and attempt to convert it into a human readable format, pseudocode

Decompilers are very beneficial for understanding parts of iOS we can't directly see.

## Conclusion

Hopefully this post has given you a good overview on the different tools and concepts we'll be using in tweak dev.
In the next post I'll go through setting up these tools ready to write our first tweak!
