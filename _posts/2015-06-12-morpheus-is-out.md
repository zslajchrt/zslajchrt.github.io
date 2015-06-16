---
layout: post
title: Morpheus Is Out
comments: true
permalink: morpheus-is-out
---

After six month of avid work on **Morpheus** I have finally reached the *beta stage*.  [Morpheus](https://github.com/zslajchrt/morpheus) is a Scala extension for
a **type-safe multi-dimensional morphing** of objects in Scala.

In other words, *Morpheus* provides a set of macros serving as a replacement for the `new` keyword. These macros not only can do everything what the `new` does, but they go beyond the traditional concept of object instantiation.

In contrast to `new`, which is used to instantiate an object composed of one or more types, Morpheus allows creating an object, called a *morph*, which is an instance of one of the set of predefined shapes called the *morh model*. 

```java
class PrettyContactPrinter implements Contact, ContactPrinter {
  // using the delegate pattern
}

ContactPrinter cp = new PrettyContactPrinter(new OfflineContact("John", "Doe", "john.doe@gmail.com"));
contact.printContact(out)
```

```scala
// using mixins
val cp = new OfflineContact("John", "Doe", "john.doe@gmail.com") with PrettyContactPrinter
contact.printContact(out)
```

```scala
// a two-dimensional morph model (Contact, ContactPrinter)
val contact = compose[(OfflineContact or OnlineContact) with (RawContactPrinter or PrettyContactPrinter)].~
contact.printContact(out)
...
val morphStrategy = promote[RawContactPrinter]
// morphStrategy activates RawContactPrinter
contact.remorph(morphStrategy)
contact.printContact(out)
```

A *morph* can be further reshaped to another shape from the morph model while preserving its identity, i.e. the reference to the object remains same.

Furthermore, morphs can be decomposed and the resulting *fragments* can be used to compose another morph.

the `new` evolution





Scala 11.x only

Used technologies: Scala compiler plugin, Scala macros, ASM, reflection
