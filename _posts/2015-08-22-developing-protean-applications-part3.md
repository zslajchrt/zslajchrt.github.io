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

The following diagram depicts sample objects with various "traits" from the protodata
domain.

Figure 2: Protodata sample model
<div>
<img src="http://zslajchrt.github.io/resources/scannerModel.png" width="560" />
</div>

#####Modeling Multidimensional Data Without Traits

Modeling in multidimensional objects in languages, which do not include traits
or similar concepts, may become rather difficult and the resulting model may not
appropriately represent reality. For example in Java, which has no equivalent
to traits, we have to implement the item's dimensions by composition. It follows
that for each dimension there will be one property in the `Items` class (`material`, `shape`).

Figure 3: No-traits diagram
<div>
<img src="http://zslajchrt.github.io/resources/itemNoTraits.png" width="500" />
</div>

Considering such a no-trait model, a new item can be created as follows.

```java
  Item item = new Item(10.1, 20.4, 3.1, new Metal(), new Cylinder(0.3, 10.32));
```

However, such a composition comes with some serious design flaws. In Java, if one
needs to know what an object is, (s)he uses the `instanceof` operator.
For example, if an instance of `Item` is tested whether it is `Thing` or `Material`
by this operator, the result will always be true by definition, since according
to the model, every item is an instance of `Thing`, `Shape` and `Metal`,
unless it is null. On the contrary, testing the item for being `Baggage` will
always be false.

```java
  boolean isThing = item instanceof Thing;       // always true
  boolean isMaterial = item instanceof Material; // always true
  boolean isBaggage = item instanceof Baggage;   // always false
```

It is clear that such a type examination is redundant and does not reveal any
additional information about the item's character. On the other hand, what we
could wish to find out, by means of the "what-it-is" `instanceof` operator, for
example, is whether the item is metal or not, since the item may or may not
have such a trait. It would by natural to express the condition as follows:

```java
  boolean isMetal = item instanceof Metal; // always false, even if the item is metal
```

Unfortunately, this statement will be always false, since the type system would have
to consult somehow the item' state to determine the "real" type of the item.

The problem is rooted in the model itself. The item's real character is not
completely carried by the item itself, but also by the wrapped instances.
It looks as if the item suffers from some kind of type schizophrenia, since its
identity is scattered across three instances: the item itself, and the material
and shape delegatees.

Actually, this is just one of several problems arising when models use delegation
or composition (e.g. composite, decorator, adapter, state, strategy, proxy, bridge).

The problem of object schizophrenia is discussed in more detail [here](http://users.jyu.fi/~sakkinen/inhws/papers/Sekharaiah.pdf).

#####Modeling Multidimensional Data With Traits

Traits can be used to model dimensions of an object. Typically, each dimension (material)
is represented by one abstract trait (`Material`). Such an abstract trait is the base for
other concrete traits (`Metal`, `Paper`) corresponding to "values" in the dimension (metal, paper).

The concept of traits is dealt with in detail here ([Groovy](http://docs.groovy-lang.org/latest/html/documentation/core-traits.html), [Scala](http://www.scala-lang.org/old/node/126)).

In contrast to the no-trait model, here the `Thing` type and the two dimensions
are separated. The composition is deferred until the moment of the creation of a new
item.

Figure 4: Traits diagram
<div>
<img src="http://zslajchrt.github.io/resources/itemTraits.png" width="560" />
</div>

The following Scala code shows how such a creation with the deferred composition can
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

The absence of the type schizophrenia in models is the key prerequisite for
a type-safe mapping between two distinct multidimensional domain models, as
shown later.

#####Modeling Multidimensional Data With Static Traits in Scala

Before going further, we should closer examine how actually a new multidimensional
instance is created from external data in Scala. The concept of static traits
seems to fit well to this multidimensional problem, since the individual dimensions
can be modeled by them.

Static traits are applied to target types at compile-time, which contributes
to the robustness of the code. The compiler checks, among other things, whether
the resulting composite classes are complete (i.e. all abstract members are
implemented) and that the used traits are applied to correct types.

When instantiating a multidimensional type, we must specify a list of traits used
for the instantiation by selecting one concrete trait (e.g. `Paper`, `Rectangle`)
for each dimension trait (`Material`, `Shape`) in accordance with the input data.
This list of traits represents a point in the multidimensional space given by
the dimensions and the traits in the list corresponding to the point's coordinates.

There is an important limitation, however, stemming from Scala's strong static
type system: traits can be specified only as a part of the class declaration.
This fact actually rules out any step-by-step imperative way to build the list
of traits from the input data (i.e. the builder pattern). Instead, the initialization
procedure must handle all possible combinations of concrete dimension traits.
I will illustrate the problem in the following example.

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
of lines (boilerplate) grows exponentially with the number of dimensions.
Unfortunately, there is currently no practical solution in Scala to cope with this problem.

Let us have a closer look at this problem. It is desirable that every item
provides information about what it is through the type system only and
not from the item's state. In other words this information must be encoded
in the **class** of which the item is an instance. It follows that there must be
as many classes in the system as the number of all possible forms which
an item can assume.

Next, there are only two ways to define a class. Either it is compiled from its
definition in the source code, or it is generated. Application developers normally
use the former way only, while the latter is there for frameworks.

If we momentarily forget the possibility to generate classes then the source
code must inevitably contain all possible composite class declarations allowed
by the domain model. Such explicit declarations can assume basically two forms.

The concrete one (using Scala notation):

```scala
class ThingPaperRectangle extends Thing with Paper with Rectangle
```

or an anonymous one, which is part of the initialization statement:

```scala
new Thing with Paper with Rectangle
```

In any case, for all coordinates in the multidimensional space there is exactly
one distinct composite class declarations.

On one hand we need composite classes since their instances carry complete information about
themselves (what they are). On the other hand, using the standard means of statically
typed languages leads to the excessive amount of boilerplate and large number
of classes.

#####Solutions of the Exponential Growth

The exponential growth of code results from the inability to apply the concrete
dimension traits in a step-by-step manner analogous to how instances are created
by the builder pattern [LINK]. Before creating an instance, the builder must be
configured in several steps according to the context and the input data.
This configuration directly or indirectly specifies the parts from which
the builder will compose the instance.

Such a step-by-step approach effectively eliminates the exponential growth, because
the number of statements is proportional to the number of dimensions (see below).

There are basically two types of such a multidimensional builder. The first one
generates a composite class according to the dimension configuration. The resulting
object is then instantiated from this generated class. The other type instantiates
a preliminary instance from the common base class. This instance is then decorated by
the selected concrete dimension traits.

While the first approach generating a composite class would require that
the developer use some rather exotic low-level tools for byte-code generation,
the second approach can be very easily employed with the help of a suitable dynamic language.
Therefore, in this paragraph we will examine this second approach, based on
the application of dynamic traits on an existing instance.

#####Modeling Multidimensional Data With Dynamic Traits in Groovy

Groovy is a dynamic language which also supports some optional static language
features, like a compile-time type check. It also contains traits that can
be applied on both types and instances. To solve the exponential growth problem
we will try to rewrite the previous code by means of Groovy's dynamic traits in
a builder-like manner.

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

It is clear that using dynamic traits and the step-by-step extensions resolved the problem
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
there are still a couple of serious problems resulting from the step-by-step
approach as well as from the dynamic nature of the language and its weak type system.

*  Incompleteness: The configuration procedure may forget to select a trait
for some dimension
*  Redundancy: The `withTraits` method can possibly add two mutually exclusive traits from
the same dimension to the builder.
*  Missing Dependencies: A trait from one dimension may depend on another trait
from another dimension. The configuration procedure must take these inter-trait
dependencies into account, because it is out of the capabilities of dynamic languages.
This responsibility makes the possibility fragile and open to inconsistencies.
*  Ambiguous Dependencies: The configuration procedure must also guarantee
that there is no ambiguity in the dependencies.
*  Since the above-mentioned responsibilities are solved on the application level
and not on the platform level, the dynamic traits approach tends to developing
unmaintainable code.

#####Modeling Multidimensional Data By Generated Classes w. Run-Time Checks

There still remains one possibility how to instantiate an instance of
a multidimensional entity: to generate the composite class at run-time.
Class generation is usually chosen as a last resort to solve problems
in applications. It is rather an extreme low-level technique requiring
experienced developers and which should be used mostly in frameworks or platforms.

Let us assume that there is a framework whereby it is possible to create a
class builder, which can be configured to build a class composed from selected
traits. The resulting class is then used to create the final instance. In contrast
to the dynamic trait approach, which creates a preliminary instance on which it then applies
the selected traits, the class generating approach first creates the class
composed of the selected traits, from which the final instance is created.

The following code shows how such a class builder could be created.

```scala
  val classBuilder = new ClassBuilder[Thing with Material with Shape]()
```

The type parameter of the builder's constructor specifies the common type for
all possible combinations of traits, which is `Thing with Material with Shape`.
This composite type will also be type of the instance created from the generated
class.

The builder's `addTrait` method serves to register a selected concrete trait.
The method also accepts a function for the initialization of the trait when
the resulting object is instantiated from the generated class.

The `Thing` trait is present in every item, so the builder is always configured
to use `Thing` along with the trait initialization function.

```scala
  classBuilder.addTrait[Thing](thing => {
    thing.x = event.get("thing").get("x")
    thing.y = event.get("thing").get("y")
    thing.z = event.get("thing").get("z")
  })
```

Next, the procedure handles the `Material` dimension. According to the existence
of `metal` or `paper` keys in the input data, the builder is configured for
the corresponding trait.

```scala  
  if (event.containsKey("metal")) {
    classBuilder.addTrait[Metal]
  } else if (event.containsKey("paper")) {
    classBuilder.addTrait[Paper]
  }
```

The `Shape` dimension is handled similarly, except the additional
trait initialization function passed to `addTrait`, by means of which
the trait's properties will be initialized from the input data upon
the instantiation.

```scala
  if (event.containsKey("rectangle")) {
    classBuilder.addTrait[Rectangle](rect => {
      rect.width = event.get("rectangle").get("width")
      rect.height = event.get("rectangle").get("height")
    })
  } else if (event.containsKey("cylinder")) {
    classBuilder.addTrait[Cylinder](cyl => {
      cyl.height = event.get("cylinder").get("height")
      cyl.radius = event.get("cylinder").get("radius")
    })
  }
```

Now it is possible to build the class and use it to create the new item instance.

```scala
  val itemClass: Class[Thing with Material with Shape] = classBuilder.buildClass
  val item: Thing with Material with Shape = item.newInstance
```

All problems associated with the dynamic traits mentioned in the previous paragraph
may be solved by the framework at run-time, but not at compile-time, because of
the imperative character of the configuration. Thus the conclusion is that:

*  The generated classes approach would be a progress in modeling multidimensional
objects in comparison to the dynamic traits. So far it is the best technique.
*  It improves the run-time part only. It would be desirable, however, to perform all
those necessary checks at compile-time.


#####Modeling Multidimensional Data By Generated Classes w. Compile-Time Checks

In this paragraph we will attempt to come up with an improvement of the previous
class generator, which would allow moving the run-time consistency checks of
the generated class to compile-time.

The previous class builder is very constrained in terms of static consistency
checks primarily because there is no static model or schema for the generated
classes. The only information available at compile-time is the type of the resulting
instance, specified as the type parameter of the builder's constructor:

```
Thing with Material with Shape
```

This type is actually the lowest upper type bound of all possible trait combinations (LUB).
From the LUB the compiler can infer the abstract dimensions, however, it cannot
infer the concrete traits constituting these dimensions. The choice, which
concrete traits come into consideration when building a class, is the task for the developer,
not the compiler. It follows that the developer must pass this information to the compiler
declaratively so as to allow a compile-time verification of multidimensional
class models.

It follows that we have to introduce some declarative construct to allow
a specification of multidimensional structures. On a statically typed
platform such as Scala, it is the most natural to express the multidimensional
structure by special type.

The principal idea is to substitute the abstract dimension traits in the type parameter
in the builder constructor with a list of possible concrete traits. The traits
in the list will be delimited by `or` to emphasize the mutually exclusive character of the traits in the list.

```
Thing with (Metal or Paper) with (Rectangle or Cylinder)
```

The previous type could be treated as a boolean formula, which can also be expressed in
the *conjunctive norm form* as follows:

```
(Thing with Metal with Rectangle) or
(Thing with Metal with Cylinder) or
(Thing with Paper with Rectangle) or
(Thing with Paper with Cylinder)
```

Such a type actually determines all possible forms of the generated composite class.
Since this information is encoded in a type, it is also available to the compiler,
which can use to perform the necessary consistency checks.

So, the new builder would be created by the following statement:

```scala
  val classBuilder = new ClassBuilder[Thing with (Metal or Paper) with (Rectangle or Cylinder)]()
```

It is assumed that there would be a compiler extension that could handle this
special type and could also build a model, from which all possible trait compositions
could be derived.

Once the compiler extension has all trait compositions, it can take one by one
and verify whether all traits in a given composition satisfy the rules such as
completeness, unambiguity etc.

The model serves primarily to check the consistency of all trait compositions.
There must be another concept, however, through which the developer
can determine, which trait composition will be used to compose the generated class.

This concept is called class builder strategy and its primary task is to
select the right trait composition from the model. An new instance of such
a strategy could be obtained from the class builder instance as follows:

```scala  
  val classBuilderStrategy = classBuilder.newClassBuilderStrategy()
```

The strategy contains `selectTrait` method through which the developer
specifies hints to the strategy, which traits must be in the resulting trait
composition.

It is not necessary to provide a hint for the `Thing` traits, since, according
to the model, it must be present always by default.

The trait hints for the material dimension can handled in the following way:

```scala
  if (event.containsKey("metal")) {
    classBuilderStrategy.selectTrait[Metal]
  } else if (event.containsKey("paper")) {
    classBuilderStrategy.selectTrait[Paper]
  }
```

And the shape dimension can be handled similarly:

```scala
  if (event.containsKey("rectangle")) {
    classBuilderStrategy.selectTrait[Rectangle](rect => {
      rect.width = event.get("rectangle").get("width")
      rect.height = event.get("rectangle").get("height")
    })
  } else if (event.containsKey("cylinder")) {
    classBuilderStrategy.selectTrait[Cylinder](cyl => {
      cyl.height = event.get("cylinder").get("height")
      cyl.radius = event.get("cylinder").get("radius")
    })
  }
```

The resulting instance is created by calling `newInstance` on the class builder
and passing the configured strategy.

```scala
  val item: Thing with Material with Shape = classBuilder.newInstance(classBuilderStrategy)
```

The item's type is the LUB of the multidimensional model, however, its
concrete type can be determined by `isInstanceOf`

```scala
  val isPaperRectangle = item.isInstanceOf[Paper with Rectangle]
```

or by the `match` block

```scala
item match {
  case pr: Paper with Rectangle =>
    ...
  case mc: Metal with Cylinder =>
    ...
}
```

This analysis indicates that all run-time checks performed by the previous version
of the class builder could be theoretically performed at compile-time in
compiler extension. Such a statically typed class builder would give developers
a powerful tool for type-safe processing of multidimensional objects.


###Summary

* The traditional approach to model multidimensional data objects uses
the composition and delegation, eventually adaptation, patterns.
The composition produces an object (item) wrapping other objects (material, shape),
which can assume various forms. The number of wrapped objects corresponds to
the number of dimensions. For each dimension the top object exposes a corresponding
interface by means of delegation.

* Delegation hides the real shape (character, type) of an object. We cannot
determine from the top object's type that it is a rectangular paper, for instance.
Instead, we just find out that the item is something of some shape and material.
To determine the real type, one cannot use a single  `instanceof` operator.
Instead, he must resort to examining the object's attributes, i.e. the state,
holding references to the wrapped objects (`getMaterial()`, `getShape()`) and
additionally apply `instanceof` for each wrapped instance.

* The character of a composite object is scattered across the object and its
components. The result is type schizophrenia.

* Traits are a natural concept for modeling multidimensional objects, which
helps preserve full object's type information and avoid type schizophrenia.

* Modeling multidimensional objects by means of static traits (Scala) leads
to exponential explosion of code making the static traits practically
unusable.

* One solution to the exponential growth could be to use a dynamic language
such as Groovy with a capability to extend object at run-time. This approach
however also suffers from a couple of serious issues as shown later in this
chapter.

* Another solution could be to create the instance from a generated class composed
of the selected traits. However, without a new declarative language construct for specifying
multidimensional models such a class builder could perform the important consistency
checks at run-time only.

* If the type system is extended in such a way that it allows declaring optional
types (union-like), the developer can specify all possible trait compositions,
i.e. the model, by means of a type expression analogous to boolean expression. This
type expression can be processed by the compiler extension and provide all necessary
checks at compile-time.
