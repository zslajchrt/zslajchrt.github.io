---
layout: post
title: Developing Protean Application With Morpheus - Part 3
comments: true
permalink: developing-protean-applications-part3
---

###Protodata

Protodata can be described as a collection of diverse data objects having no immediate
and meaningful commercial use or value.

It comes into existence usually as a by-product of some business activity,
such as journal logs, but it may also be leftover of some shut down project or
data acquired as part of some acquisition.

Protodata may also be deliberately gathered for a longer period of time without
any specific vision of its future use besides the owner's hope that such a large
amount of data may become a gold-mine in the future.

Applying the knowledge matrix introduced in the previous chapter, the level of the data owner's
knowledge at this stage would correspond to categories *Known-Knowns* or *Known-Unknowns*,
when the known value is insignificant or there is considerable risk embedded
in the attempt to sell data possibly containing delicate information.

The lack of a usage means that there is **no context** within which the data objects
in the protodata would yield some commercially or otherwise valuable information.
The objects can be treated only as bare wrappers of intrinsic properties (i.e.
properties that an object has of itself, independently of other objects,
including its context. [1](https://en.wikipedia.org/wiki/Intrinsic_and_extrinsic_properties))

In order to utilize the protodata, it must be put into a novel context. It can
be achieved, for instance, by combining the protodata with other, often apparently
unrelated, data or with newly emerged information services.

The owner's effort to find a new context for his protodata can go in two directions:
To try to find
   1. something valuable
   2. anything valuable

The first way is less difficult, since it evaluates existing contexts against the protodata.
For example, a competition has developed a successful service using data that is
somehow similar to ours. So we will examine our protodata whether it is able to
yield similar, or possibly better, information if combined with other available
data sources. This situation corresponds to the *Unknown-Knowns* epistemological
category, since the owner knows the value of the information he would like to retrieve
from the protodata, however, it is still unknown, it there is such information.

The second way is significantly more difficult, since its goal is to
invent a new and original context, which would become a market differentiator.
This approach usually requires a longer research, while the results are highly uncertain.
The knowledge category in this case corresponds to *Unknown-Unknowns*, since
not only does the owner not know if there is something unusually interesting, but
even if there were something interesting he would not know its value.

In any case, if some vital context is found it also comes with a new domain model.
Although protodata may be accompanied by some metadata, i.e. having its own
domain model - *proto-domain*, the context domain model may be so structurally
and semantically different from the proto-domain that a direct mapping from
the proto-domain to the context domain can be difficult.

If such a situation occurs, the usual solution is to transform and normalize
the protodata in such a way that the mapping will be easier. However, this
approach has several drawbacks.

First, transformations and normalizations are lossy processes by its nature.
It follows that during such processes some information, which could be potentially
used in a future development of the application, will be inevitably lost. In
such a case, the processes will have to be redesigned to provide the required
additional data and the protodata will have to re-processed. And it can, of course,
consume a lot of resources and time.

Second, if the protodata has character of a stream of events then any additional
pre-processing could be a source of undesired delays.

Third, when another useful context is found, new transforming routes will have
to be established, which will put another burden on the infrastructure.
Moreover, reusing the processes already established for the existing contexts,
may not be possible because of the diverse nature of contexts and their domain models.

Fourth, in the course of time when more contexts are implemented the system of
processes will tend to become unmaintainable and resistant to refactoring.

It seems that the only way to bypass the above-mentioned issues is to try to
**map directly** the context domain onto the proto-domain, regardless of the diverse character
of the two domains, and avoid any intermediary processing.

Furthermore, since the type systems of advanced programming languages are
powerful enough to grasp the complex nature of domain objects, domain models
should model the objects by type as much as possible and not by state.

If domain objects are modeled by state then the objects' character,
i.e. what the objects are, is retrieved from the properties of the objects. For
example in Java, more complex objects with multidimensional character must
be modeled by means of delegation, which is de-facto a modeling by state, as shown
in the next paragraph.

If domain objects are modeled by type then the objects' character is retrieved
from their type. This approach has many benefits, since a lot of responsibilities
can be delegated on the type system. Additionally, if the language is static,
many possible errors can be discovered at compile-time.

Languages like Scala or Groovy [LINK] come with the concept of trait, which
is very useful to model diverse nature of complex objects by type.

The main goal of this work (Morpheus) is to prove that it is possible to construct
a domain mapper as an extension of the language platform. In other words, the mapper
becomes integrated with the language itself in contrast to other mapping tools [LINK],
which are built on top the language. It will also be shown that such a mapper
inherits the nice properties of static and statically typed languages such as
type-safety and early discovery of errors, i.e. the mapping schemas are validated
at compile-time.

In the following paragraph I will present an example on which I would like
to illustrate the described problem as well as to sketch its solution.
The protodata in the example is data collected from a fictional airport scanner,
which is able to recognize objects in baggage.

####Utilizing Airport Scanner Logs

For the sake of simplicity and clarity let us assume this fictional narrative:

An airport outsources its security check procedure to a company operating
high-tech scanners, which are able to recognize three and two-dimensional objects
in baggage. After each scan the scanner creates a record describing the recognized
objects in a baggage. The record is provided with some metadata and sent to
a server at the mother company's premises through a wide-area network connection.

The company's server receives a potent stream of such events from a number of
installations at several airports. The event in the stream is stored into a clustered
database, where it remains for one year. The stored events are not processed in
any way, the only reason for storing the events follows from the contract with
the airports, which obliges the company to keep the record for one year due to
forensic purposes.

A major limitation of the scanner is that it is able to classify baggage items
only by their geometrical properties and material. The recognized item is described
as a plain geometrical object and not as a wallet, shaver etc.

The contract itself does not forbid the company to use the scanner data commercially
under the condition that such a usage reveals no private and sensitive information.

Such data is a typical example of protodata. The owning company comes to a conclusion
that the data cannot be easily utilized as such due its primitive character and
possible infringement of the contract by selling it in the raw form to another
subject, which could combine it with its own data and possibly misuse the insights
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
and material properties. A sample of a scan event is on Figure 1.

/// Figure 1: JSON samples

The sample suggests that the structure of the scanned item has two degrees of freedom:
shape and material. These two dimensions can be described as traits of the scanned item.

Using term *trait* seems quite adequate since some programming languages, such as
Scala, implement the concept of trait (or mixin), which perfectly suits to modeling
multidimensional objects.

The following diagram depicts the classes and traits involved in the protodata
domain.

/// Figure 2: Protodata sample model
<div>
<img src="http://zslajchrt.github.io/resources/scannerModel.png" width="560" />
</div>

Modeling in multidimensional objects in languages, which do not include traits
or similar concepts, may become rather difficult and the resulting model may not
appropriately grasp the reality. For example in Java, which has no equivalent
to traits, we would have to substitute traits with interfaces and delegation.

/// Figure 3: No-traits diagram
<div>
<img src="http://zslajchrt.github.io/resources/itemNoTraits.png" width="500" />
</div>

Considering this model, a new item can be created as follows.

///Code 1
```java
  Item item = new Item(10.1, 20.4, 3.1, new Metal(), new Cylinder(0.3, 10.32));
```


However, this substitution comes with some serious design flaws. According to
the model, every item is an instance of `Thing` and two types `Shape`
and `Metal`. In Java, if one needs to know what an object is, (s)he uses
the `instanceof` operator. For example, if an instance of `Item` is tested
whether it is `Thing` or `Material` by this operator, the result will always be true.
On the other hand, testing the item for being `Baggage` will never be true.

///Code 2
```java
  boolean isThing = item instaceof Thing;       // always true
  boolean isMaterial = item instaceof Material; // always true
  boolean isBaggage = item instaceof Baggage;   // always false
```

A problem arises when one wishes to check whether the item is metal. The following
statement will always be false regardless of the possibility that the item may
be, or rather represent, a metal item.

///Code 3
```java
  val isMetal = item instaceof Metal; // always false, even if the item is metal
```

By using `instanceof` we cannot find out more details about the real character
of the item.

The problem is rooted in the model itself. The item's real character is not
completely carried by the item itself, but also by the wrapped instances.
It looks as if the item suffers from some kind of schizophrenia, since its
identity is scattered across three instances: the item itself, and the metal and
shape delegatees.

Actually, this is just one of more problems arising when models use delegation
and similar patterns such as decorator, adapter, state, strategy, proxy, bridge.
[LINK: Patterns]

The problem of object schizophrenia is discussed in more detail here [LINK: Object schizophrenia].

On the other hand, let us see how the model looks if the language offers traits [LINK].
In contrast to the no-trait model, here the `Thing` type and the two dimensions
are separated. The composition is deferred until the moment of the creation of a new
item.

/// Figure 4: Traits diagram
<div>
<img src="http://zslajchrt.github.io/resources/itemTraits.png" width="560" />
</div>

The following code shows how such a creation with the deferred composition can
look like.

///Code 4
```scala
  val item = new Thing with Metal with Cylinder {
    val x = 10.1
    val y = 20.4
    val z = 3.1
    val height = 0.3
    val radius = 10.32
  }
```

Then the item can be tested with `isInstaceOf`, which is analogous to Java's
`instanceof`.

///Code 5
```scala
  val isMetal = item.isInstaceOf[Metal]         // true
  val isCylinder = item.isInstaceOf[Cylinder]   // true
```

Now, it is possible to determine the exact type, i.e. what the object exactly is. The reason
is that the identity is no longer scattered among more instances but the item only.

Furthermore, one can also use composite types to test instances:

///Code 6
```scala
  val isMetal = item.isInstaceOf[Metal with Cylinder] // true or false
```

The absence of the object schizophrenia in models is the key feature when
the direct mapping between two distinct multidimensional domain models, as
shown later in this text [WHERE].

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

Before going further, we should closer examine how actually a new multidimensional
instance is created from external data in Scala.

Let us assume that the external data for the item is stored in a dictionary-like
object `event`. In order to create a new item then for every
`(material, shape)` combination there must be a dedicated test in the creation code,
as shown in the following snippet.

///Code 7
```scala
  val x = event.get("thing").get("x")
  val y = event.get("thing").get("y")
  val z = event.get("thing").get("z")
  val item = if (event.contains("metal") && event.contains("cylinder")) {
    new Thing(x, y, z) with Metal with Cylinder {
      val height = event.get("cylinder").get("height")
      val radius = event.get("cylinder").get("height")
    }
  } else if (event.contains("metal") && event.contains("rectangle")) {
    new Thing(x, y, z) with Metal with Cylinder {
      val width = event.get("rectangle").get("width")
      val height = event.get("rectangle").get("height")
    }
  } else if (event.contains("paper") && event.contains("cylinder")) {
    new Thing(x, y, z) with Metal with Cylinder {
      val height = event.get("cylinder").get("height")
      val radius = event.get("cylinder").get("height")
    }
  } else if (event.contains("paper") && event.contains("rectangle")) {
    new Thing(x, y, z) with Metal with Cylinder {
      val width = event.get("rectangle").get("width")
      val height = event.get("rectangle").get("height")
    }
  }
```

It is obvious that such an instantiation is unsustainable since the number
of lines (boilerplate) grows exponentially with the number of dimensions. Unfortunately,
there is currently no practical solution in Scala to cope with this
problem. It is, after all, a phenomenon tightly connected with static
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

In order to find a solution we must not only take into account
the generation, but also consider some extension of the type system.

The solution stems from the idea to express the multidimensionality of a domain
object as a type. It is actually an extension of the traditional way of expressing
object types. For example the following expression refers to a simple type:

```scala
  Thing
```

and this expression refers to a composition of three type.

```scala
  Thing with Paper with Cylinder
```

It would be nice if another type expression could be constructed to refer to a type
containing alternatives (a sort of union).

```scala
  Thing with (Paper or Metal) with (Rectangle or Cylinder)
```

The previous type expresses in one line all combinations of forms that an item
can assume. Adding a new dimension or a new type to an existing dimension does
not cause a code explosion.

To support such a kind of expression in applications, there must be some extension
in the language platform, especially in the compiler and type system sections.

The following code shows how such a complex type could be used to instantiate
an item (the code uses the real *Morpheus* code):

```scala
  val itemKernel = compose[Thing with (Paper or Metal) with (Rectangle or Cylinder)]
  val item = itemKernel.morph(new ItemBuilder(scanEvent)) // ItemBuilder determines the final form of the item

  assert(true, item.isInstaceOf[Thing with Material with Shape])

  val maybeBanknote = item.isInstaceOf[Paper with Rectangle] // true or false
```

Without delving too much into the details, the program flow can be described
this way:

First, the `compose` operator creates the so-called kernel for the type specified
in the brackets. The kernel 'knows' all the alternative forms which an item
can assume. It also holds factories for instantiating individual fragments, of
which the type is composed.

Second, the item is instantiated by invoking `morph` method on the kernel.
Since there are a number of possible alternatives, the kernel needs a companion,
which helps him to determine the right alternative. The `ItemBuilder` is such
a companion which selects the correct alternative according to the `scanEvent`
passed as the argument to its constructor.

Third, the returned object is always an instance of type `Thing with Material with Shape`,
which is the lowest common ancestor type of all alternative types.

Fourth, the item can be checked whether it is a certain combination of
concrete types by means of `isInstanceOf` method as usual.

The complete code including the initialization is discussed later.

For the sake of further analysis let us assume that at this stage the multidimensional
protodata can be modeled by means of the extended system system sketched above.

#####New Context Domain For Protodata

Now we turn our attention back to the airport scanner protodata and the effort
to find some use for it.

Let us assume that someone comes up with an idea to build an application performing
some analytics of the luggage contents and publishing the insights via a web portal
to customers, which could be for example the statistics bureau, economic chambers
or customs.

As the first analysis the application will publish the aggregate amount of cash
detected in luggage per some time period.

Such a task requires that the application be able to recognize coins and banknotes
among luggage items. Considering the scanner protodata contains information
about the shape and material only, it seems inevitable to use other source to
detect reliably coins and banknotes. Such a source can be a database of
international currencies containing comprehensive information about current and
historical world currencies including physical properties.

The domain model of the new context is depicted on the following diagram:

/// Figure 5: The context domain diagram
<div>
<img src="http://zslajchrt.github.io/resources/currencyModel.png" width="280" />
</div>

The basic logic of the currency flow analysis is very simple. It scans a segment
of the protodata corresponding to the selected time period and it attempts to
convert each luggage item into a currency. Such a set of currency objects is then
aggregated into a statistical report and published on the portal.

The only missing part is now the conversion of the item to a currency object. This
task is analyzed in the next paragraph.

#####Mapping the context domain on the proto-domain

The goal of the mapping is to bind the objects from the target domain to
the objects from the source domain in order to avoid any intermediary processes
normalizing and transforming the source domains objects into the target domain ones.

Furthermore, since the type systems of advanced programming languages are
powerful enough to grasp the complex nature of domain objects, domain models
should use primarily the type system to model the objects.

Here, the source domains are the scanner protodata and the currency database, while
the target domain is the currency context. The mapping declares how an object
from the target domain can be bound to objects from the source domains. Particularly,
the mapping in this example contains rules for mapping coins and banknotes to
luggage items and records in the currency database.

The mapping between the currency and luggage items domain is sketched on the following diagram.

/// Figure 6: Mapping context domain to proto-domain diagram
<div>
<img src="http://zslajchrt.github.io/resources/itemCurrencyMap.png" width="450" />
</div>

Each concrete type in the target domain is mapped to a subset of source domain
concrete types. Moreover, this mapping is *orthogonal* since all these
subsets do not overlap with one another.

Since the mapping should utilize primarily the capabilities of the underlying
type system, the rules should be declared by means of type expressions
composed of the trait types defined in the source and target domains.

Let us try to formally declare the mapping rules for coins and banknotes:

```
  Coin -> {Metal, Cylinder}
  Banknote -> {Paper, Rectangle}
```

In other words, these rules say that a coin is a metal cylinder and a banknote
is a paper rectangle. Of course, such definitions are far from complete. Not
every metal cylinder is a coin.

This is the moment, when the auxiliary currency database is used to complete the missing
parts in the definitions.

We can declare that a metal cylinder is a coin if the currency database contains
a coin record, whose physical properties correspond to that of the metal cylinder,
of course, within some predefined margin of error.

To reflect these additional constraints the Coin mapping rule can be completed as follows:

```
  Coin -> {m: Metal, c: Cylinder, ce: CoinExemplar(radius = c.diameter/2, thickness = c.height)}
```

The rule for Coin is now a composition of three components and can be seen as
a template for valid Coin instances demanding existence of the three constituting
components.

If an item being mapped to a coin is a metal cylinder, then the existence of
the first two components is fulfilled automatically. However, the existence of
the third component must be confirmed by a lookup in the currency database.

Every component is annotated with an identifier, which can be used in expressions
constraining property values in other components such as `radius` and `thickness` in
CoinExemplar.

*Note: Since the resulting object is a composition of three existing object from the source
domains there is no lost of information, even if the target object exposes only
some of it. The complete information is encapsulated in the target object and
some future minor changes in the domain code can make accessible.*

Once the existence of all components is confirmed, the new object can be made
up of its components. The constituting components are deprived of their identity
in behalf of the newly composed entity. It is necessary in order to prevent
the object schizophrenia.

Unfortunately, the described process of object identity deprivation is impossible
or very difficult to carry out, especially in statically typed languages. [LINKS, DCI etc.]
Once an object is instantiated it keeps its identity until its destruction. There
is no concept of 'identity deprivation' or 'getting rid of identity'.
The only way to compose an object of other already existing objects is to use
delegation, which immediately leads to object schizophrenia and thus to the inability
to determine the object's character from its type.

Therefore, to make the direct mapping based on type modeling feasible it is
necessary to rethink the process of instantiation in programming languages and
and come up with some alternative.

#####Alternative Instantiation Process

The traditional approach in all class-oriented languages follows this instantiation
schema:

```
   class -> class instance
```

In other words, the class precedes its instances, which sounds logical.

However, it is possible to consider other, a more general schema, which introduces
the concept of *fragment*. The fragment can be seen as a weaker version of
the class. It may or may not be concrete, however, even if it is abstract, it can
be instantiated in contrast to an abstract class. Such abstract fragment instances
can have state but cannot be used independently.

The alternative instantiation schema employs fragments and is expressed in the following
causal chain:

```
   fragments -> fragment instances -> composite class -> composite class instance
```

The schema can interpreted as follows: Fragments and possibly also their
instances exist in the system even before there exists any class. All classes
in the system are compositions of one or more fragments and all instances of
such classes are compositions of corresponding fragment instances.

Let us examine the behavior of the proposed mechanism in the simplest case:

```
   x = instantiate X("Hello")
```

The `instantiate` procedure starts with looking for fragment `X` in the system. The system
responds by returning a factory for obtaining an instance of `X`, otherwise an exception
is raised. Then the factory returns the instance, which is immediately returned as
the result of the procedure. The procedure assumes that the compiler has already
done all necessary checks so that it can simply return the fragment instance.
In this case there is no need for generating a composite class since there is
only one fragment involved in the instantiation.

Let us examine now a more complex example.

```
  xy = instantiate X("Hello"), Y
```

Now the procedure starts looking for fragments `X` and `Y` and obtains their instances.
Since now the target class is a composition of `X` and `Y`, the class loader is asked
to load and possibly generate the class. Then an instance of this class is composed
from the instances of fragments `X` and `Y`. Since there is no constructor argument
list specified for Y it can be interpreted as a signal to not create a new instance
and instead to use a cached fragment instance (e.g. a singleton).

Note: Fragments should be considered having no identity until after the instantiation
procedure finishes.

#####Summary

A new context must be discovered in order for "useless" protodata to yield some value

The data objects in the protodata may manifest the so-called *multidimensional polymorphism*.
Multidimensional polymorphism means that the set of all forms, which an object can assume,
is equivalent to a cartesian product of the so-called dimensions. A dimension
represents an abstract trait of the object and consists of types implementing
this trait.

The new context may found by evaluating the protodata against a new data source - the auxiliary data

The domain of the new context and the proto-domain can be structurally very
distant, possibly orthogonal.

The protodata and the auxiliary data must be somehow adapted for the new context domain

Transformation and normalization are lossy processes, thus they remove data, which
the application could use in the future. Next, such a processing may cause delays
and put additional burden on the infrastructure. In the course of time the transforming
processes tend to become unmaintainable.

The solution is to map the new context domain directly to the source domains and
to compose the target domain objects of the objects from the source domains.

The mapping and composition should be the task for the so-called mapper.

The mapper should be domain-agnostic, i.e. its functionality should
not be principally driven by the knowledge of the participating domains.
In other words, the mapper should be able to map one domain onto another only
by means of the underlying language platform, especially the type-system.

There more reasons for it:
   * **Stability** - The character or meaning of domain entities is more stable than the structure
   of individual entities. It follows that if the meaning and character is expressed
   by types and not by state, then possible structural changes in the domain model
   should not affect the mapping schema.
   * **Type safety** - The type system verifies the consistency of the mapping rules,
   i.e. that the types involved in the mapping can be mapped and composed as intended.
   It can detect missing parts and makes the mapping schema robust against changes
   in the domain models.
   * **Early error discovery** - If a static language is used then the consistency
   check and the verifications are carried out at compile-time. It follows that
   possible errors and inconsistencies in the mapping may be caught early.

Java's type system does not provide sufficient means to express the real character
of objects by type. Therefore, designers must resort to the modeling of the object
character by state (e.g. delegation) whereby they incorporate object schizophrenia
into the model.

The concept of traits as implemented in Scala or Groovy fits very well to
the needs of such a mapper.

However, using traits to express multidimensional objects leads to the explosion
of class declarations and boilerplate code.

This problem can be resolved by extending the type system so that the multidimensional
space of all possible object forms can be grasped by a special type expression.

The result of a mapping is a new target domain object composed of the source domains
objects. In order to avoid object schizophrenia the source objects must be
deprived of their identity in behalf of the new target domain object.

It is practically impossible to carry out the identity deprivation in the
current statically typed languages. Therefore an alternative object instantiation
procedure, which generalizes the current one, must be incorporated to the
used language platform.
