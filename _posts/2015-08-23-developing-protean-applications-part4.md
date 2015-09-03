---
layout: post
title: Developing Protean Application With Morpheus - Part 4
comments: true
permalink: developing-protean-applications-part4
---

###New Context Domain For Protodata

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
historical world currencies including physical properties. (Note: There is a limitation
in this scenario, since all U.S. banknotes have the same dimensions. The database
could not recognize individual dollar banknotes just on the basis of their size).

The domain model of the new context is depicted on the following diagram:

Figure 5: The context domain diagram
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
Here, the implementor of the `Banknote` trait can be sure, that the mapping code
will be executed in the context of an object implementing both `Rectangle` and `Paper`
traits. It follows that it is possible to safely access the members of
`Rectangle` and `Paper` traits.

the major part of the mapping is left to the platform, resp. to the type system.

On the contrary, if the item does carry any type information about its "traits",
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

it is the developer who must guarantee that the item is the right one.

#####Mapping Currencies To Items by Type

In this case the source domains are the scanner protodata and the currency database, while
the target domain is the currency context. The mapping declares how an object
from the target domain can be bound to objects from the source domains. Particularly,
the mapping in this example contains rules for mapping coins and banknotes to
luggage items and records in the currency database.

The mapping between the currency and luggage items domain is sketched on the following diagram.

Figure 6: Mapping context domain to proto-domain diagram
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

#####Dynamic Traits Mapping Issues

In this paragraph we will try if the above-mentioned problems disappears if
we use dynamic traits.

The semantics of the the `mapCurrencyOnItem` method is the same as in
the previous paragraph except the return type, which is not optional.
The method simply returns `null` in case it cannot construct a currency object.

```groovy
public Currency mapCurrencyOnItem(Thing item) {
    if ((item instanceof Metal) && (item instanceof Cylinder)) {
        CoinExemplar coinEx = currencyDb.findCoin(2 * item.radius, item.height)
        if (coinEx != null) {
            Coin coin = item.withTraits(Coin, CoinExemplar)
            // we cannot avoid the copying of the coinEx's state
            coin.adoptExemplar(coinEx)
            return coin;
        } else {
            return null;
        }
    } else if ((item instanceof Paper) && (item instanceof Rectangle)) {
      ...
    }
}
```

As shown in the previous section, on a platform with dynamic traits it is possible
to extend objects in such a way that the extending objects can completely
preserve the type information carried by the extended objects. It permits us
to determine what the object is by means of `instanceof`.

Thus, the `mapCurrencyOnItem` method determines whether the item is a metal
cylinder by two invocations of `instanceof`. Then the method asks the currency database
if such a coin is registered. If so, it uses `withTraits` to create a new proxy
object from the item object. The new proxy object has all the item's traits plus
`Coin` and `CoinExemplar`.

Unfortunately, using `withTraits` helped us only in part. We avoided copying
the item's state to the new instance, but there is the `coinEx` object left,
whose state must be copied to the new instance. This is a conceptual problem
stemming from the nature of traits, which are primarily supposed to extend
the target object's behavior, not its state. Yet if a trait has some state,
such a state has no meaning out of the target object's scope. In the light
of this concept it makes a perfect sense for Groovy to provide the `withTraits`
special method, which is invoked on the target object accepting a list of
trait types, not trait "instances".

Unfortunately, the conclusion is that the use of dynamic traits does not
solve the problem with copying state between instances as such.

#####Summary

* The data objects in the protodata manifest the so-called *multidimensional polymorphism*.
Multidimensional polymorphism means that the set of all forms, which an object can assume,
is equivalent to a cartesian product of the so-called dimensions. A dimension
represents an abstract trait of the object and consists of types implementing
this trait.

* The target domain and the source domains are structurally very distant,
practically orthogonal. The protodata (items) and the auxiliary data (currency database)
must be somehow adapted for the new context domain.

* Transformation and normalization are lossy processes, thus they remove data, which
the application could use in the future. Next, such a processing may cause delays
and put additional burden on the infrastructure. In the course of time the transforming
processes tend to become unmaintainable.

* The solution is to map the new context domain directly to the source domains and
to compose the target domain objects of the objects from the source domains.

* Such a mapping should be domain-agnostic, i.e. its functionality should
not be principally driven by the knowledge of the participating domains.
In other words, the mapper should be able to map one domain onto another only
by means of the underlying language platform, especially the type-system.

There more reasons for it:

  * **Stability**: The character or meaning of domain entities is more stable than the structure
   of individual entities. It follows that if the meaning and character is expressed
   by types and not by state, then possible structural changes in the domain model
   should not affect the mapping schema.
  * **Type safety**: The type system verifies the consistency of the mapping rules,
   i.e. that the types involved in the mapping can be mapped and composed as intended.
   It can detect missing parts and makes the mapping schema robust against changes
   in the domain models.
  * **Early error discovery**: If a static language is used then the consistency
   check and the verifications are carried out at compile-time. It follows that
   possible errors and inconsistencies in the mapping may be caught early.

* Java's type system does not provide sufficient means to express the real character
of objects by type. Therefore, designers must resort to the modeling of the object
character by state (e.g. delegation) whereby they incorporate object schizophrenia
into the model.

* The concept of traits as implemented in Scala or Groovy fits very well to
the needs of such a mapping.

* However, using static traits to express multidimensional objects leads to the explosion
of class declarations and the code as such. Using dynamic traits solves this problem.

* Neither static nor dynamic traits solve the problem of copying states between
source and target instances during the mapping.

#####Source Code

The following [link](https://github.com/zslajchrt/morpheus-tutor/tree/master/src/main/scala/org/cloudio/morpheus/scanner)
refers to a source code containing a runnable implementation of the Scaner protodata
case study developed by means [Morpheus](https://github.com/zslajchrt/morpheus).
