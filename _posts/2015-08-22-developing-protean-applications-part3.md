---
layout: post
title: Developing Protean Application With Morpheus - Part 3
comments: true
permalink: developing-protean-applications-part3
---

###Utilizing Airport Scanner Logs

For the sake of simplicity and clarity let us assume this fictional narrative:

An airport outsources its security check procedure to a company operating
high-tech scanners, which are able to recognize three and two-dimensional
objects in baggage. After each scan the scanner creates a record describing the recognized
objects in a piece of baggage. The record is provided with some metadata and sent to
a server at the mother company's premises through a wide-area network connection.

The company's server receives a potent stream of such events from a number of
installations at several airports. The events in the stream are stored in a clustered
database, where they remain for one year. The stored events are not processed in
any way, the only reason for storing the events follows from the contract with
the airports, which obliges the company to keep the records for one year due to
forensic purposes.

A major limitation of the scanner is that it is able to classify baggage items
only by their geometrical properties and material. The recognized item is described
as a plain geometrical object and not as a wallet, shaver etc.

The contract itself does not forbid the company to use the scanner data commercially
under the condition that such usage reveals no private and sensitive information.

Such data is a typical example of protodata. The owning company comes to a conclusion
that the data cannot be easily utilized as such due to its primitive character and
possible infringement of the contract by selling it in raw form to another
entity, which could combine it with its own data and possibly misuse the insights
by revealing sensitive information.

The owner realizes that without discovering a new context the scanner data is
virtually of no use.

But before such a context is discovered and applied, an overview of the scanner
protodata should be given.

#####Scanner Protodata Overview

The scan event consists of two parts: metadata and data. The metadata contains
the time of the scan, the serial number of the scanner and the id of the
baggage. The data part consists of a list of items recognized in the baggage.
Each item contains space coordinates relative to the baggage and geometrical
and material properties. A sample of a scan event is in Figure 1.

Figure 1: JSON samples

```json
{
  "id": 8488484,
  "scanTime": "2013-05-23T00:00:00Z",
  "scannerId": 78700,
  "items": [
    {
      "id": 0,
      "shape": "cylinder",
      "material": "metal",
      "x": 234.87,
      "y": 133.4,
      "z": 12.94,
      "diameter": 13.45,
      "height": 0.45,
      "reflexivity": 0.8
    },
    {
      "id": 1,
      "shape": "rectangle",
      "material": "paper",
      "x": 673.87,
      "y": 394.4,
      "z": 132.93,
      "width": 13.45,
      "height": 8.21,
      "reflexivity": 0.2
    }
  ]
}
```

The sample suggests that the structure of the scanned item has two degrees of freedom:
shape and material. These two dimensions can be described as traits of the scanned item.

Using term *trait* seems quite adequate since some programming languages, such as
Scala, implement the concept of trait (or mixin), which perfectly suits to modeling
multidimensional objects.

The following diagram depicts the classes and traits involved in the protodata
domain.

Figure 2: Protodata sample model
<div>
<img src="http://zslajchrt.github.io/resources/scannerModel.png" width="560" />
</div>

Modeling in multidimensional objects in languages, which do not include traits
or similar concepts, may become rather difficult and the resulting model may not
appropriately represent reality. For example in Java, which has no equivalent
to traits, we would have to substitute interfaces and delegation for traits.

Figure 3: No-traits diagram
<div>
<img src="http://zslajchrt.github.io/resources/itemNoTraits.png" width="500" />
</div>

Considering this model, a new item can be created as follows.

```java
  Item item = new Item(10.1, 20.4, 3.1, new Metal(), new Cylinder(0.3, 10.32));
```


However, this substitution comes with some serious design flaws. According to
the model, every item is an instance of `Thing` and two types `Shape`
and `Metal`. In Java, if one needs to know what an object is, (s)he uses
the `instanceof` operator. For example, if an instance of `Item` is tested
whether it is `Thing` or `Material` by this operator, the result will always be true.
On the other hand, testing the item for being `Baggage` will never be true.

```java
  boolean isThing = item instanceof Thing;       // always true
  boolean isMaterial = item instanceof Material; // always true
  boolean isBaggage = item instanceof Baggage;   // always false
```

A problem arises when one wishes to check whether the item is metal. The following
statement will always be false regardless of the possibility that the item may
be, or rather represent, a metal item.

```java
  boolean isMetal = item instanceof Metal; // always false, even if the item is metal
```

By using `instanceof` we cannot find out more details about the real character
of the item.

The problem is rooted in the model itself. The item's real character is not
completely carried by the item itself, but also by the wrapped instances.
It looks as if the item suffers from some kind of schizophrenia, since its
identity is scattered across three instances: the item itself, and the metal and
shape delegatees.

Actually, this is just one of several problems arising when models use delegation
and similar patterns such as decorator, adapter, state, strategy, proxy, bridge.

The problem of object schizophrenia is discussed in more detail [here](http://users.jyu.fi/~sakkinen/inhws/papers/Sekharaiah.pdf).

On the other hand, let us see how the model looks if the language offers traits ([Groovy](http://docs.groovy-lang.org/latest/html/documentation/core-traits.html), [Scala](http://www.scala-lang.org/old/node/126)).
In contrast to the no-trait model, here the `Thing` type and the two dimensions
are separated. The composition is deferred until the moment of the creation of a new
item.

Figure 4: Traits diagram
<div>
<img src="http://zslajchrt.github.io/resources/itemTraits.png" width="560" />
</div>

The following code shows how such a creation with the deferred composition can
look.

```scala
  val item = new Thing with Metal with Cylinder {
    val x = 10.1
    val y = 20.4
    val z = 3.1
    val height = 0.3
    val radius = 10.32
  }
```

Then the item can be tested with `isInstanceOf`, which is analogous to Java's
`instanceof`.

```scala
  val isMetal = item.isInstanceOf[Metal]         // true
  val isCylinder = item.isInstanceOf[Cylinder]   // true
```

Now, it is possible to determine the exact type, i.e. what the object exactly is. The reason
is that the identity is no longer scattered among more instances but is carried solely by the item.

Furthermore, one can also use composite types to test instances:

```scala
  val isMetal = item.isInstanceOf[Metal with Cylinder] // true or false
```

The absence of the object schizophrenia in models is the key prerequisite for
functional mapping between two distinct multidimensional domain models, as
shown later in this text.

The next two pieces of code illustrate the problem. Both determine
whether the item represents a rectangular paper. If so, the banknote database
is requested to try to find the corresponding banknote.

```java
  if ((item.material instanceof Paper) && (item.shape instanceof Rectangle)) {
    return banknotesDb.findBanknote((Rectangle)item.shape);
  }
```

```scala
  item match {
    case prItem: Paper with Rectangle =>
      banknotesDb.findBanknote(prItem)
    case _ => None  
  }
```

The Java mapper must evaluate the types of the two item's attributes `material` and
`shape` in order to determine the "type" of the item itself. Thus, this mapper
must inevitably understand the structure of the item.

On the other hand, the Scala mapper makes do with the item itself and does not
need any knowledge of the item's internal structure. Since `prItem` is an instance
of `Paper with Rectangle` it is possible to pass it to `findBanknote`, which
accepts instances of `Rectangle`.

The difference becomes very important when building a tool for domain mapping.
Ideally, all information about what the objects in the domains **are** should
be retrieved from the **type system** of the language only with no evaluation
of the objects' state at all. In such a case the mapper can be considered an
extension of the language's type system (and the compiler). It would have a
important consequence that domain mapping schemas could be type-checked for
correctness and type safety at compile-time.

#####Modeling multidimensional in Scala (static traits)

Before going further, we should closer examine how actually a new multidimensional
instance is created from external data in Scala. The concept of static traits
seems to fit well to this multidimensional problem, since the individual dimensions
can be modeled by them.

Typically, the initialization procedure prepares a list of traits by selecting
one trait from each dimension (e.g. `Paper`, `Rectangle`) according to the input
data. This list of traits represents a point in the multidimensional space given
by the dimensions and the traits in the list correspond to the point's coordinates.

There is an important limitation, however, stemming from Scala's strong static
type system: traits can be specified only as a part of the class declaration.
This fact actually rules out any step-by-step imperative way to build the list
of traits from the input data (i.e. the builder pattern). Instead, the initialization
procedure must handle all possible combinations of traits. I will illustrate the
problem in the following example.

Let us assume that the external data for the item is stored in a dictionary-like
object `event`. In order to create a new item then for every
`(material, shape)` combination there must be a dedicated test in the creation code,
as shown in the following snippet.

```scala
  val x = event.get("thing").get("x")
  val y = event.get("thing").get("y")
  val z = event.get("thing").get("z")
  val item = if (event.containsKey("metal") && event.containsKey("cylinder")) {
    new Thing(x, y, z) with Metal with Cylinder {
      val height = event.get("cylinder").get("height")
      val radius = event.get("cylinder").get("radius")
    }
  } else if (event.containsKey("metal") && event.containsKey("rectangle")) {
    new Thing(x, y, z) with Metal with Cylinder {
      val width = event.get("rectangle").get("width")
      val height = event.get("rectangle").get("height")
    }
  } else if (event.containsKey("paper") && event.containsKey("cylinder")) {
    new Thing(x, y, z) with Metal with Cylinder {
      val height = event.get("cylinder").get("height")
      val radius = event.get("cylinder").get("radius")
    }
  } else if (event.containsKey("paper") && event.containsKey("rectangle")) {
    new Thing(x, y, z) with Metal with Cylinder {
      val width = event.get("rectangle").get("width")
      val height = event.get("rectangle").get("height")
    }
  }
```

It is obvious that such an instantiation is unsustainable since the number
of lines (boilerplate) grows exponentially with the number of dimensions. Unfortunately,
there is currently no practical solution in Scala (and any other statically typed language)
to cope with this problem. It is, after all, a phenomenon tightly connected with static
type systems.

Let us have a closer look at this problem. It is required that every item
must provide information about what it is through the type system only and
not from the item's state. In other words this information must be encoded
in the class of which the item is an instance. It follows that there must be
as many classes in the system as the number of all possible forms which
an item can assume.

Next, there are only two ways to define a class. Either it is compiled from its
definition in the source code, or it is generated. Application developers normally
use the former way only, while the latter is there for framework developers.

If we momentarily forget the possibility to generate classes then the source
code must inevitably contain all possible composite class declarations allowed
by the domain model. Such explicit declarations can assume basically two forms.

The concrete one (using Scala notation):

```scala
class ThingPaperRectangle extends Thing with Paper with Rectangle
```

or an anonymous one, which is a part of the initialization statement:

```scala
new Thing with Paper with Rectangle
```

In any case, for all coordinates in the multidimensional space there is exactly
one distinct composite class declarations.

On one hand we need composite classes since their instances carry complete information about
themselves (what they are). On the other hand, using the standard means of statically
typed languages leads to the excessive amount of boilerplate and large number
of classes.

#####Modeling multidimensional in Groovy (dynamic traits)

Since static traits in Scala leads to the exponential growth of code when
attempting to model a multidimensional data, we can try a dynamic approach.

Groovy is a dynamic language which, however, supports some optional static language
features, like a compile-time type check. It also contains traits that can
be applied on both types and instances. To solve the exponential growth problem
we will try to rewrite the previous code by means of the dynamic traits.

The common data of the item is represented by trait `Thing`, which contains the
coordinates (x, y, z) of the item in the luggage. The following code creates
the item object implementing the `Thing` trait. The item object is then initialized
from the input data.

```groovy
def item = new Object() as Thing

Map thingData = event.get("thing")
item.x = thingData.get("x")
item.y = thingData.get("y")
item.z = thingData.get("z")
```

Now we can use the dynamic traits and extend the item step-by-step by the right traits.
We use `withTraits` method invoked on the item object to extend it by the selected trait.
The selected trait is immediately initialized from the input data.

```groovy
// Shape dimension
if (event.containsKey("cylinder")) {
    item = item.withTraits(Cylinder)
    Map cylinderData = event.get("cylinder")
    item.height = cylinderData.get("height")
    item.radius = cylinderData.get("radius")
} else if (event.containsKey("rectangle")) {
    item = item.withTraits(Rectangle)
    Map rectangleData = event.get("rectangle")
    item.height = rectangleData.get("height")
    item.width = rectangleData.get("width")
}

// Material dimension
if (event.containsKey("metal")) {
    item = item.withTraits(Metal)
} else if (event.containsKey("paper")) {
    item = item.withTraits(Paper)
}
```

It seems that using dynamic traits and the step-by-step extensions resolved the problem
with the exponential growth. The number of conditions is proportional to
the number of dimensions and not to the number of all possible combinations.

The resulting object also carries all type information about itself. It can
be tested by the `instanceof` operator.

```groovy
def isThing = item instanceof Thing
def isRectangle = item instanceof Rectangle
def isPaper = item instanceof Paper

if (isThing && isRectangle && isPaper)
    println(/${item.x}, ${item.y}, ${item.z}, ${item.width}, ${item.height}/)
```

Although this solution based on using the dynamic traits seems to be
perfect, because it allows us to capture the multidimensional character of data
by traits, preserves the types and prevents the code from exponential growth,
there are still a couple of serious problems connected to the dynamic
nature of the language and its weak type system. As shown later in this chapter,
modeling complex multidimensional domains with the dynamic traits on a platform
with a weak type system tend to unmaintainable code.

###Summary

* The traditional approach to model multidimensional data objects uses
the composition and delegation, eventually adaptation, patterns.
The composition produces an object (item) wrapping other objects (material, shape),
which can assume various forms. The number of wrapped objects corresponds to
the number of dimensions. For each dimension the top object exposes a corresponding
interface by means of delegation.

* Delegation hides the real shape (character, type) of an object. We cannot
determine from the top object's type that it is a rectangular paper. Instead,
we always find out that it is a something of some shape and material.
To determine the real type, one cannot use a single  `instanceof` operator.
Instead, he must resort to examining the object's attributes, i.e. the state,
holding references to the wrapped objects (`getMaterial()`, `getShape()`) and
additionally apply `instanceof` for each wrapped instance.

* The character of a composite object is scattered across the object and its
components. The result is the so-called object schizophrenia.

* Traits are a natural concept for modeling multidimensional objects, which
helps preserve full object's type information and avoid object schizophrenia.

* Modeling multidimensional objects by means of static traits (Scala) leads
to exponential explosion of code making the static traits practically
unusable. It can be solved either by using a dynamic language with traits or
by extending a static language.

* One solution to the exponential growth could be to use a dynamic language
such as Groovy with a capability to extend object at run-time. This approach
however also suffers from a couple of serious issues as shown later in this
chapter.
