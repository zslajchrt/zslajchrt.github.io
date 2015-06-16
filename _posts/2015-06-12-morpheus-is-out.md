---
layout: post
title: Morpheus Is Out
comments: true
permalink: morpheus-is-out
---

After six month of avid work on **Morpheus** I have finally reached the *beta stage*.  [Morpheus](https://github.com/zslajchrt/morpheus) is a Scala extension for
a **type-safe morphing** of objects in Scala.

In other words, *Morpheus* provides a set of macros serving as a replacement for the `new` keyword. These macros not only can do everything what the `new` does, but they go beyond the traditional concept of object instantiation.
These macros extend the `new`swhich is used to compose and instantiate objects. the instances called *morphs* can be further reshaped or re-composed from the so-called fragments.

the `new` evolution

Scala 11.x only

Used technologies: Scala compiler plugin, Scala macros, ASM, reflection
