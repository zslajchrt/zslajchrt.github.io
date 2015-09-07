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

The relation between the currency and luggage items domain is sketched on the following diagram.

Figure 6: Mapping context domain to proto-domain diagram
<div>
<img src="http://zslajchrt.github.io/resources/itemCurrencyMap.png" width="450" />
</div>

Each concrete type in the target domain is mapped to a subset of the source domain
concrete types. Moreover, this mapping is *orthogonal* since all these
subsets do not overlap with one another.

The only missing part is the construction of currency objects from the source objects,
i.e. from appropriate items and currency database records. This task is called
*mapping* and is analyzed in the next paragraph.

#####Mapping Target Objects To Source Ones

The purpose of the mapping is to bind objects from the target domain directly to
the objects from the source domain in order to avoid any intermediary processes
normalizing and transforming the source domains objects into the target domain ones.
Such processes often discard some source data, which could potentially be utilized
in subsequent transformations.

The goal is to construct a lossless mapping. In other words, the target objects
preserve both data and behavior of the source objects.

A target object constructed by binding it directly to its source objects allow tracking
its origins, which is as desirable property in the case when the target object becomes
a source in another mapping. Such a secondary mapping may exploit the information about the
origin of the source (then-target) object to perform a finer-grained binding.

In the following paragraphs we will attempt to design such a lossless mapping procedure.

#####Mapping Currencies To Items by Properties

The following listing is an example of a low-level lossless mapping in Scala. The source and target
objects are plain maps, whose values can be other maps or scalar values. The result
of the mapping procedure is a map representing a scanned banknote, whose single property
`isExported` combines properties from the item and the banknote record in the database.
The result also wraps the source objects (item, exemplar) so as not to loose
the track of its origin.

```scala
  def makeCurrency(item: Map[String, Object]): Option[Map[String, Object]] = {
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
        case Some(banknote) =>
          val isExported = banknote.get("issuingCountry") == item.
            get("luggage").asInstanceOf[Map[String, Object]].
            get("scanner").asInstanceOf[Map[String, Object]].
            get("location").asInstanceOf[Map[String, Object]].
            get("country")
          val banknote = Map("type" -> "scanned-banknote",
                             "isExported" -> isExported,
                             "item" -> item,
                             "currency" -> banknote)
          Some(banknote)  
      }
    } else ... // a similar code for a coin
  }
```

The method first checks whether the `shape` and `material` dimension properties
hold rectangle, respective paper data. If so, the method retrieves the width,
height and color from the two components. Then the currency database is queried
for a banknote having the corresponding properties. It a banknote exemplar is found,
a new map representing the banknote is created.

Although this mapping is lossless, it suffers from several flaws, such as:

* It is too low-level, domain objects are represented as maps, not as domain types.
It is prone to type mismatch errors.
* There is no behavior on the domain objects (maps). Target objects should be able
to inherit some behavior from the source objects. For example, the `Shape` trait
could define a method calculating the area (or surface) of the shape. It makes
good sense for banknotes and coins as the target objects to inherit this method.
* There must be some convention on how the source data are embedded into
the target objects. Such a convention makes the mapping subsystem very proprietary.
* It is too verbose, too much boilerplate.

The absence of types and behavior is evidently the most annoying issue of
this property-based approach. Let us try therefore to evaluate an approach
using properties and types of the domain objects.

#####Mapping Currencies To Items by Properties and Static Traits

The mapping procedure in this paragraph combines the properties of the domain objects
as well as their types. The item contains two properties, `shape` and `material`,
representing the two independent dimensions of the item. The mapping procedure
first reads the wrapped objects held in these two dimension properties and
examines their concrete types. In other words, in order to find out what the
item really is we have to look into its properties and determine the types of
their values.

The individual types of the domain objects and the are represented by Scala static traits.

The application of static traits to target classes takes place at compile-time.
The compiler checks among other things whether the resulting composite classes
are complete (i.e. all abstract members are implemented) and that the used traits
are applied to correct classes

Let us assume first, that the multidimensional character of the item is modeled
by means of composition. This approach would have to be used on platforms without
dynamic traits (such as Scala or Java) because of the problems with the exponential
growth of code described previously.

```scala
trait Item {
  def shape: Shape
  def material: Material
  ...
}
```

The source domain is described in the previous section, so here we focus on
the description of the currency database types and the target domain types.

The currency database has a simple model consisting of two types of currency records:
`Banknote` and `Coin`, both having the same ancestor `Currency`.

```scala
trait Currency {
  def currencyValue: BigDecimal
  def currencyCode: String
  def issuingCountry: String
}
trait Banknote extends Currency
trait Coin extends Currency
```

The `Currency` trait consists of a couple of basic properties, while the other
traits may carry some specific properties. (They are omitted since they are not important for
the purpose of this example.)

The target types mimic the structure of the currency database domain. The `ScannedCurrency`
trait represents a currency found in a luggage and its single property `isExported`
combines data from the two source domains. A currency is considered *exported*
if the currency's issuing country is the same as the country where the item was
scanned.

```scala
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
```

Now, let us turn our attention to the mapping method `makeCurrency` using
the above-mentioned types.

```scala
def makeCurrency(item: Item): Option[ScannedCurrency] = {
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

The method must first check the types of the two dimensions `shape` and `material`
held as properties in the item. Then the values are type-casted to the concrete
types to retrieve the width, height of the rectangle and the paper color.

Then the database is asked to find the corresponding banknote. If it is found,
a new instance of `ScannedBanknote` is created, which implements both `Banknote`
and `Item` traits by delegation.

At first sight, the code looks tidier than the property-based one. However, there
are still some issues:

* Nothing prevents `ScannedBanknote` from being instantiated with wrong
(i.e. not banknote-like) item. The reason is that the proper composition
of `ScannedBanknote` with luggage items must be guaranteed by the developer and
not by the platform. The developer does it by examining the item's state, while
the platform would do it by examining the types.

* Another issue relates to the delegation: on each consecutive mapping the item
instance looses some behavior because not all interfaces implemented by
the source object must be implemented by the target object. The item instances also
sinks one delegation level down gradually becoming unreachable.

These two findings can be formulated more generally:

* A multidimensional source object modeled by composition makes subsequent compositions
prone to type inconsistencies.
* The construction of target objects by means of delegations tends to lose behavior
and type.


#####Mapping Currencies To Items by Static Traits Only

In this paragraph we will examine a purely static type-based approach to mapping
target objects on the source ones by means of static traits.

Since one of the problems described in the previous paragraph was caused by
composition, i.e. by the wrapped dimensions in the item object, in this example
we will try to model the multidimensionality of items (no type schizophrenia)
by static traits (Scala).
It follows that there will be no properties in `Item` representing
the two dimensions. Instead, the dimensions become part of the item's type
expressed by means of traits.

Note: Let us temporarily forget the problem with the exponential growth of code
linked to using statically specified traits as described in the previous section.

Since now the shape and material dimension are part of the item's type, we can
refine the binding self-type in the `ScannedBanknote` so as to refer
`Banknote with Rectangle with Paper` instead of `Banknote with Item`.

```scala
trait ScannedBanknote extends ScannedCurrency {
  this: Banknote with Rectangle with Paper =>
}
```

This refinement actually solves the problem with the possibility of combining
`ScannedBanknote` with inappropriate item. The type system ensures that `ScannedBanknote`
can be instantiated only with an item implementing `Rectangle` and `Paper`.

Thanks to the presence of dimension types in the item's type the mapping
method can perform the test for paper rectangles directly on the item. Instead of
using `isInstanceOf[Rectangle with Paper]` we use Scala's `match` block, which
is more suitable for this purpose:

```scala
def makeCurrency(item: Item): Option[ScannerCurrency] = {
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

Unfortunately, the new instance cannot avoid the delegation. The conclusion is
that event if we suppressed the problem of the exponential growth of code,

* The composition issue is over, `ScannedBanknote` can be accompanied by paper rectangles only
* The delegation issue still persists, i.e. behavior and type degrade on every successive mapping


#####Mapping Currencies To Items by Dynamic Traits

In this paragraph we will try if the above-mentioned problems disappears if
we use dynamic traits.

The semantics of the the `makeCurrency` method is the same as in
the previous paragraphs except the return type, which is not optional.
The method simply returns `null` in case it cannot construct a currency object.

```groovy
public Currency makeCurrency(Item item) {
    if ((item instanceof Paper) && (item instanceof Rectangle)) {
        Banknote banknote = currencyDb.findBanknote(item.width, item.height, item.color)
        if (banknote != null) {
            ScannedBanknote scannedBanknote = item.withTraits(ScannedBanknote, Banknote)
            // we cannot avoid the copying of the banknote's state to scannedBanknote
            scannedBanknote.adoptBanknote(banknote)
            return scannedBanknote;
        } else {
            return null;
        }
    } else ... // a similar code for a coin
}
```

As shown in the previous section, on a platform with dynamic traits it is possible
to extend objects in such a way that the extending objects can completely
preserve the type information carried by the extended objects. It permits us
to determine what the object is by means of `instanceof`.

Thus, the `makeCurrency` method determines whether the item is a paper
rectangle by two invocations of `instanceof`. Then the method asks the currency database
if such a banknote is registered. If so, it uses `withTraits` to create a new proxy
object from the item object. The new proxy object has all the item's traits plus
`ScannedBanknote` and `Banknote`.

Unfortunately, using `withTraits` helped us only in part. We avoided copying
the item's state to the new instance, but there is the `banknote` object left,
whose state must be copied to the new instance. This is a conceptual problem
stemming from the nature of traits, which are primarily supposed to extend
the target object's behavior, not its state. Yet if a trait has some state,
such a state has no meaning out of the target object's scope. In the light
of this concept it makes a perfect sense for Groovy to provide the `withTraits`
special method, which is invoked on the target object accepting a list of
trait types, not trait "instances".

The conclusion is:

* The use of dynamic traits solves the problem with losing the item's type and behavior,
because `withTraits` automatically implements all interfaces on the source object.
* The removal of the delegation was helpful in the previous issue, however it brings
another problem: the `banknote`'s state must be copied to the `scannedBanknote` instance.
The `banknote` can theoretically be an instance of an extended trait carrying additional
data. This extra state will not be copied to `scannedBanknote`, thus there is a potential
of losing some information when copying state.
* The problem of combining `ScannedBanknote` with a wrong item emerges again, because
the weak type system and the dynamic nature of Groovy does not allow checking
the inter-trait relationships.


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
by means of the underlying language platform, especially the type-system. There
more reasons for it:
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
  * **Clean code**: Since many of responsibilities can be delegated on the underlying
  platform, the code contains only the necessary stuff.

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
