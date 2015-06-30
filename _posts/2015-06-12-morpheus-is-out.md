---
layout: post
title: Morpheus Is Out
comments: true
permalink: morpheus-is-out
---

After six months of dilligent work on **Morpheus** I have finally reached the *beta stage* of development.  [Morpheus](https://github.com/zslajchrt/morpheus) is a Scala extension for
a **type-safe multi-dimensional morphing** of objects in Scala.

In other words, *Morpheus* provides a set of macros serving as a replacement for the `new` keyword. These macros can not only do everything that the `new` does, but they go beyond the traditional concept of object instantiation.

In contrast to `new`, which is used to instantiate an object composed of one or more types, Morpheus allows the creation of an object, called a *morph*, which is an instance of one of the set of predefined shapes called the *morph model*.

A *morph* can be further reshaped to another shape from the morph model while preserving its identity, i.e. the reference to the object remains same.

```scala
// a two-dimensional morph model (Contact, ContactPrinter)
val contact = compose[(OfflineContact or OnlineContact) with
                    (RawContactPrinter or PrettyContactPrinter)].~
contact.printContact(out)

val morphStrategy = promote[RawContactPrinter]
// morphStrategy activates RawContactPrinter
contact.remorph(morphStrategy)
contact.printContact(out)
```

Furthermore, morphs can be decomposed and the resulting *fragments* can be used to compose another morph.

For a quick introduction to Morpheus visit the following links:

[README](https://github.com/zslajchrt/morpheus) and the [Morpheus tutorial](https://github.com/zslajchrt/morpheus-tutor)

Currently, Morpheus is built for Scala 11.x only. It will be ported to higher versions later as soon as it becomes stable on version 11.

Morpheus would never see the light of day without the following interesting technologies: [Scala compiler plugin API](http://www.scala-lang.org/old/node/140), [Scala macros](http://docs.scala-lang.org/overviews/macros/overview.html), [Shapeless](https://github.com/milessabin/shapeless), [ASM (a bytecode manipulation library)](http://asm.ow2.org), and [Scala reflection](http://docs.scala-lang.org/overviews/reflection/overview.html).
