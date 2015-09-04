---
layout: post
title: Developing Protean Application With Morpheus - Part 8
comments: true
permalink: developing-protean-applications-part8
---

###Wrapping Up

Strong type-systems capabilities cannot be fully utilized when using composition
and delegation. These techniques are often used as a workaround, because of
the platform limitations such as a lack of dynamic features. The type system is
circumvented and application-specific methods are used to examine the character
of objects instead. A typical application hoovers somewhere between the property-
based and the type-based approaches.

In the light of the above-mentioned findings,... strong

```
  Item ::= thing Material Shape
  Material ::= paper metal other
  Shape ::= rectangle cylinder other
```

```
  thing metal cylinder
  thing metal rectangle
  thing metal other
  thing paper cylinder
  thing paper rectangle
  thing paper other
  thing other cylinder
  thing other rectangle
  thing other other
```


```scala
  def toBanknote(item: Map[String, Object]): Option[Map[String, Object]] = {
    val shape = item.get("shape")
    val material = item.get("material")
    if (shape == null || !"rectangle".shape.get("type") ||
        material == null || !"paper".material.get("type")) {
      None
    } else {
      val w = shape.asInstanceOf[Map[String, Object]].get("width");
      val h = shape.asInstanceOf[Map[String, Object]].get("height");
      val c = material.asInstanceOf[Map[String, Object]].get("color");

      currencyDb.findBanknote(w, h, c) match {
        case None => None
        case Some(exemplar) =>
          val banknote = Map("type" -> "banknote",
                             "value" -> exemplar.value,
                             "position" -> item.get("position"),
                             "item" -> item,
                             "exemplar" -> exemplar)
          Some(banknote)  
      }
    }
  }
```

Preserves both, "types" and data

```scala

  trait Item {
    def shape: Shape
    def material: Material
    def luggage: Luggage
    ...
  }

  trait Currency {
    def currencyValue: BigDecimal
    def currencyCode: String
    def issuingCountry: String
  }
  trait Banknote extends Currency
  trait Coin extends Currency

  trait ScannedCurrency {
    this: Currency with Item =>

    val isExported: Boolean = issuingCountry == luggage.scanner.location.country
  }

  trait ScannedBanknote extends ScannedCurrency {
    this: Banknote with Item =>
  }

  trait ScannedCoin extends ScannedCurrency {
    this: Coin with Item =>
  }

  def toBanknote(item: Item): Option[ScannedCurrency] = {
    if (item.shape.isInstanceOf[Rectangle] && item.material.isInstanceOf[Paper]) {
      val rect = item.shape.isInstanceOf[Rectangle]
      val paper = item.material.isInstanceOf[Paper]
      val w = rect.width;
      val h = rect.height;
      val c = paper.color;

      currencyDb.findBanknote(w, h, c) match {
        case None => None
        case Some(banknote) =>
          val scannedBanknote = new ScannedBanknote with Banknote with Item {
            val itemDlg = item
            val banknoteDlg = banknote
            // Delegating methods ...
          }
          Some(scannedBanknote)
      }

    } else ... // a similar code for a coin
  }
```

```scala
  scannedCurrency match {
    case detectedBanknote: ScannedCurrency with Banknote =>
      new Abc with ScannedCurrency with Banknote {
        val detDlg = detectedBanknote
      }
      // the new instance no longer implements Item
      // the item is now wrapped twice and becomes unreachable
  }
```

* Composition issue: Nothing prevents `ScannedBanknote` with Banknote with Item` from being
instantiated with wrong (i.e. not banknote-like) item since the right item is determined
by examining the item's state and not type.
* Delegation issue: On each consecutive mapping the item instance
  * sinks one delegation level down gradually becoming unreachable.
  * looses some behavior, because not all interface implemented by the source object
  are implemented by the target one.


We cannot trace the origin of the banknote object. Both "type" and some data are lost
in the translation.


```scala
  trait ScannedCurrency {
    this: Currency with Item =>

    val isExported: Boolean = issuingCountry == luggage.scanner.location.country
  }

  trait ScannedBanknote extends ScannedCurrency {
    this: Banknote with Rectangle with Paper =>
  }

  def getRectangleAreaAndPaperColor(item: Item): Option[ScannerCurrency] = {
    item match {
      case paperRect: Rectangle with Paper =>
        currencyDb.findBanknote(paperRect.width, paperRect.height, paperRect.color) match {
              case None => None
              case Some(exemplar) =>
                val scannedBanknote = new ScannedBanknote with Banknote with Rectangle with Paper {
                  val paperRectDlg = paperRect
                  val banknoteDlg = banknote
                  // Delegating methods ...
                }
                Some(scannedBanknote)
        }
      case _ => None
    }
  }
```

* The composition issue is over, ScannedBanknote can be accompanied by paper rectangles only
* The delegation issue still persists

```scala
  trait Banknote {
    this: Rectangle with Paper with BanknoteExemplar =>
    ...
  }

  def toBanknote(item: Item): Option[Banknote] = {
    item match {
      case pr: Rectangle with Paper =>
        currencyDb.findBanknote(pr.width, pr.height, pr.color) match {
              case None => None
              case Some(exemplar) =>
                Some(item with exemplar with Banknote)  
        }
      case _ => None
    }
  }
```
Preserves both, types and data


It preserves the type but potentially looses some data



```scala
  def getRectangleAreaAndPaperColor(item: ParsedObject): Option[Float, Int] = {
    item match {
      case Item(Rectangle(w, _), Paper(c)) => Some((w, c))
      case _ => None
    }
  }

  trait AAA {
    this: Item(Rectangle(w, h), Paper(c)) => ???
  }

  case class AAA(r: Item(Rectangle(w, ), Pap)) ???

```

```java
  void processItem(<Thing & (Paper | Metal | Other) & (Rectangle | Cylinder | Other )> item) {
    ...
  }
```

####Type Preservation
Why?


####Multidimensionality

TODO: What is it?
An instance may assume any of the set of predefined forms. Each form corresponds
to a point in a multidimensional space, where each dimension is represented by
an abstract "trait" (material) and values in the dimension are "concrete" traits (paper, metal ...).

modeling multidimensional data, combinatorial explosion of class declarations,
multidimensional polymorphism -> a need to describe all combinations in one expression

####Metamorphism

TODO: What is it? It

####Multidimensional Metamorphism

TODO: What is it?

a loss-less extension of types, the information about the types constituting
an object's class should percolate through all abstraction layers,
from the persistence layer, through the business layer to the presentation layer,
loose coupling,

cloning object state

the client is tightly coupled with a concrete implementation of a general interface:
to find out the type of the underlying entity, to determine the implemented
interfaces (isXXX methods)

The functionality of AlternatingUserMail should be provided by the platform.

####A Vision of New Platform

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

  assert(true, item.isInstanceOf[Thing with Material with Shape])

  val maybeBanknote = item.isInstanceOf[Paper with Rectangle] // true or false
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

#####Object Identity Deprivation

Once the existence of all components is confirmed, the new object can be made
up of its components. The constituting components are deprived of their identity
in behalf of the newly composed entity. It is necessary in order to prevent
the object schizophrenia.

Unfortunately, the described process of object identity deprivation is impossible
or very difficult to carry out, especially in statically typed languages ([here]((http://www.artima.com/articles/dci_vision.html)) or [here](http://www.sitepoint.com/dci-the-evolution-of-the-object-oriented-paradigm/)).
Once an object is instantiated it keeps its identity until its destruction. There
is no concept of 'identity deprivation' or 'getting rid of identity'.
The only way to compose an object of other already existing objects is to use
delegation, which immediately leads to object schizophrenia and thus to the inability
to determine the object's character from its type.

Therefore, to make the direct mapping based on type modeling feasible it is
necessary to rethink the process of instantiation in programming languages and
and come up with some alternative.


#####Mapping

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

The rules can easily express in Scala by traits and the self-type.

```scala
trait Coin {
  this: Metal with Cylinder =>
...
}

trait Banknote {
  this: Paper with Rectangle =>
...
}
```

This is the moment, when the auxiliary currency database is used to complete the missing
parts in the definitions.

We can declare that a metal cylinder is a coin if the currency database contains
a coin record, whose physical properties correspond to that of the metal cylinder,
of course, within some predefined margin of error.

To reflect these additional constraints the Coin mapping rule can be completed as follows:

```
  Coin -> {m: Metal, c: Cylinder, ce: CoinExemplar(radius = c.diameter/2, thickness = c.height)}
```

The rule for `Coin` is now a composition of three components and can be seen as
a template for valid `Coin` instances demanding existence of the three constituting
components. Every component is annotated with an identifier, which can be used in expressions
constraining property values in other components such as `radius` and `thickness` in
CoinExemplar.

This extended rule can also be expressed by a trait as follows:

trait Coin {
  this: Metal with Cylinder with CoinExemplar =>

  require(radius == diameter/2)
  require(thickness == height)
...
}

The trait also includes two `require` statements checking the constraints
between the `Cylinder`s and `CoinExemplar`s properties.

If an item being mapped to a coin is a metal cylinder, then the existence of
the first two components is fulfilled automatically. However, the existence of
the third component must be confirmed by a lookup in the currency database.
Of course, this last step cannot be delegated to the platform's type system and
must be done by the application itself.

The resulting object is a composition of two source objects (item and currency exemplar)
with a target trait (`Coin` or `Banknote`). Although the target object interface
provides only a view of the underlying source objects, there is, in fact, no lost
of information, neither data nor type information. The complete information is
encapsulated in the target object and some new features of the target domain
code can make use of it in the future.

In order to make the mapping easier and also to reduce the coupling between
the two domain models, the type of individual domain objects should contain
as much information about objects' character (i.e. what they are) as possible
and avoid describing the object by state (i.e. no "isABC" properties).

Then we can describe a banknote as a rectangular paper and not as a mere item.

The following listing shows how we can express the mapping between banknotes and
rectangular paper items in Scala. We use the so-called self-type, which specifies
the context for traits.

```scala
trait Banknote {
  // this self-type limits the use of this trait to instances of Rectangle and Paper only
  this: Paper with Rectangle =>

  // here it is possible to access the members of Rectangle and Paper traits
  val isValidBanknote: Boolean =
   currencyDb.findBanknote(this.width, this.height) match {
     case None => false
     case Some(banknote) => banknote.isValid
   }

   ...
}
```
The implementor of the `Banknote` trait can be sure, that the mapping code
will be executed in the context of an object implementing both `Rectangle` and `Paper`
traits. It follows that it is possible to safely access the members of
`Rectangle` and `Paper` traits.

On the contrary, if the item does not carry any type information about its "traits",
we would be limited to a too weak mapping between the banknotes and items.
We will have to resort to casting an item's property (such as shape) to the requested type
hoping that the context object (`this`) is associated with "right" item.

```scala
trait Banknote {
  this: Item =>

  val width = this.shape.asInstanceOf[Rectangle].width
  val height = this.shape.asInstanceOf[Rectangle].height

  val isValidBanknote: Boolean = ...
}
```

This evidently introduces a loophole in the design, which can become a source
of future problems. In particular, nothing prevents the creator of a `Banknote`
instance from attaching the `Banknote` trait to a wrong item.

After executing the following statement, it is guaranteed by the platform
that the banknote is an instance of both `Rectangle` and `Paper`.

```scala
  val banknote = new Thing with Paper with Rectangle with Banknote
```

On the other hand, after executing this statement:

```scala
  val banknote = new Item with Banknote
```

It is the developer who must guarantee that the item is the right one.


#####Static Traits Mapping Issues

In order to illustrate the problems connected to the use of static traits, let us
sketch the code constructing a currency object from an item.

```scala
def mapCurrencyOnItem(item: Thing): Option[Currency] = item match {

    case metCyl: Metal with Cylinder  =>
      currencyDb.findCoin(2 * metCyl.radius, metCyl.height) match {
        case None => None
        case Some(coinEx) =>
          val coin = new Metal with Cylinder with CoinExemplar with Banknote
          // How to get metCyl and coinEx into the new coin object?
          Some(coin)
      }

      case paperRect: Paper with Rectangle  =>
        ...
  }
```

The `mapCurrencyOnItem` method converts the item passed as the argument to a
currency object. We use `Option[Currency]` as the return type, which allows
returning `None` if neither coin nor banknote can be constructed from the item.

In Scala one can use the `match` pattern matcher as an elegant replacement for `isInstanceOf`.
Each `case` keyword in the matcher's body can be followed by a labeled type expression,
which selects only such objects whose type matches the type expression.

In this example there are two `case` blocks, one capturing metal cylinders and
the other capturing paper rectangles.

When a metal cylinder is caught, for example, the currency database is consulted
to return the corresponding coin exemplar. The result is processed in another
`match` block. If the exemplar is found, then the metal cylinder item is by definition
considered a coin.

At this moment we have to create the instance of `Coin`. In Scala, we have to
create a new instance composed of all necessary traits: `Metal`, `Cylinder`,
`CoinExemplar` and `Banknote`. The `Thing` is not necessary to include since it
is not referenced from any other trait.

Then we have to associate somehow the new coin object with the two source objects
`metCyl` and `coinEx`. In Scala, we have unfortunately no other option than
to copy the state of the source objects to the new object, even if we tend
to apply the `Banknote` trait directly on a composition of the two instances in
a way shown in the following state):

```scala
val coin = metCyl with coinEx with Banknote
```

The fact that we cannot avoid copying the states of the source objects to
the target one along with the exponential growth of the code when modeling
multidimensional objects are the two key issues when using static traits to
develop applications on top of multidimensional domains.
