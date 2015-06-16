---
layout: post
title: Morpheus Is Out
comments: true
permalink: morpheus-is-out
---

After six month of avid work on **Morpheus** I have finally reached the *beta stage*.  [Morpheus](https://github.com/zslajchrt/morpheus) is a Scala extension for
a **type-safe morphing** of objects in Scala.

In other words, *Morpheus* provides a set of macros serving as a replacement for the `new` keyword. These macros not only can do everything what the `new` does, but they go beyond the traditional concept of object instantiation.

In contrast to `new`, which is used to instantiate an object composed of one or more types, Morpheus allows creating an object, called a *morph*, which is an instance of one of the set of predefined shapes called the *morh model*. 

A *morph* can be further reshaped to another shape from the morph model while preserving its identity, i.e. the reference to the object remains same.

the `new` evolution

Scala 11.x only

Used technologies: Scala compiler plugin, Scala macros, ASM, reflection
