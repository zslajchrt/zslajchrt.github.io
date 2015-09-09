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

A target object constructed by binding it directly to its source objects allows tracking
its origins, which is as desirable property in the case when the target object becomes
a source in another mapping. Such a secondary mapping may exploit the information about the
origin of the source (then-target) object to perform a finer-grained binding.

In the following paragraphs we will attempt to design such a lossless mapping procedure.

#####Mapping Currencies To Items by Properties

The following listing is an example of a low-level lossless mapping in Scala. The source and target
objects are plain maps, whose values can be other maps or scalar values. The result
of the mapping procedure is a map representing a scanned currency, whose single property
`isExported` combines properties from the item and the currency record in the database.
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
their values (i.e. type schizophrenia).

The item, its dimensions (i.e. material, shape) and the dimensions' "values"
(i.e. paper, rectangle etc.) are represented by Scala static traits.

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
combines data from the two source domains: a currency is considered *exported*
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

The self-type (`=>`) of `ScannedCurrency` constrains the application of this trait
only to the compositions containing `Currency` and `Item` traits. Since any object
containing `ScannedCurrency` must also contain the two traits it is possible to
access the `Currency` and `Item` members from the body of `ScannedCurrency`.

`ScannedBanknote` constrains its application even more to objects with
the `Banknote` and `Item` traits. Similarly, `ScannedCoin` limits is use to
the context of `Coin` and `Item`.

The self-type constrains in `ScannedBanknote` and `ScannedCoin` could be even
stricter if we could replace `Item` with `Paper with Rectangle`, resp. `Metal with Cylinder`.
However, it is not possible, because of the type schizophrenia of the item.

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
included in the item as its properties. Then the values are type-casted to the concrete
types to retrieve the width, height of the rectangle and the paper color.

Then the database is asked to find the corresponding banknote. If it is found,
a new instance of `ScannedBanknote` is created, which implements both `Banknote`
and `Item` traits by delegation.

At first sight, the code looks much tidier than the property-based one. However, there
are still some issues:

* Nothing prevents `ScannedBanknote` from being instantiated with wrong
(i.e. not banknote-like) item. The reason is that the proper composition
of `ScannedBanknote` with luggage items must be guaranteed by the developer and
not by the platform. The developer does it by examining the item's state, while
the platform would do it by examining the types. It could be prevented by specifying
the self-type of `ScannedBanknote` as `Banknote with Paper with Rectangle`, but
it is not possible because of the type schizophrenia of the item.

* Another issue relates to the delegation: on each consecutive mapping the item
instance looses some behavior because not all interfaces implemented by
the source object must be implemented by the target object. The wrapped item instance also
sinks one delegation level down becoming gradually unreachable.

These two findings can be formulated more generally:

* A multidimensional source object modeled by **composition** makes subsequent compositions
prone to type inconsistencies.
* The construction of target objects by means of **delegations** tends to lose behavior
and type.


#####Mapping Currencies To Items by Static Traits Only

In this paragraph we will examine a purely static type-based approach to mapping
target objects on the source ones by means of static traits.

Since one of the problems described in the previous paragraph was caused by
composition, i.e. by the dimensions wrapped in the item object, in this example
we will try to model the multidimensionality of items by Scala static traits
(no type schizophrenia). It follows that there will be no properties in `Item` representing
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
that with the supposition of ignoring the problem of the exponential growth of code,

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
The `banknote` can theoretically be an instance of a `Banknote` subclass carrying additional
data. This extra state will not be copied to `scannedBanknote`, thus there is a potential
of losing some information when copying state.
* The problem of combining `ScannedBanknote` with a wrong item emerges again, because
the weak type system and the dynamic nature of Groovy does not allow checking
the inter-trait relationships.


#####Mapping Currencies To Items by Class Generator

So far no mapping attempt has offered an ideal solution. The Scala showed to be too
stiff while Groovy too flexible. This paragraph is dealing with an alternative
approach searching for a middle course, which would lead us to a statically checked
yet dynamic mapping.

The basic building block in such an approach is the fictional class builder presented
in the previous section. This builder is able to build consistent compositions
of traits according to the model specified as a special type expression in the builder's
constructor. The compiler is supposed to execute the consistency checks of the model
so that the builder can rely on the model's consistency at run-time.

In Scala it is possible to define type aliases by means of the so-called type
members. To make the code more maintainable and readable, we will use such type aliases
instead of the original type expressions throughout the code. The first one is
the alias of the item's multidimensional model.

```scala
type ItemModel = Thing with (Metal or Paper) with (Rectangle or Cylinder)
```

This model can be rewritten to the equivalent disjunctive form, which
shows better the four possible compositions of the traits.

```
(Thing with Metal with Rectangle) or
(Thing with Metal with Cylinder) or
(Thing with Paper with Rectangle) or
(Thing with Paper with Cylinder)
```

The item builder is created by instantiating `ClassBuilder` and specifying
the `ItemModel` type alias as the constructor's type argument and the trait
initializer as the other argument.

```scala
  val itemBuilder = new ClassBuilder[ItemModel]({
    case rect: Rectangle =>
      rect.width = event.get("rectangle").get("width")
      rect.height = event.get("rectangle").get("height")
    case cyl: Cylinder =>
      cyl.height = event.get("cylinder").get("height")
      cyl.radius = event.get("cylinder").get("radius")
  })
  val itemBuilderStrategy = new HintClassBuilderStrategy[ItemModel]()
  ... // configuring the strategy
  val itemRef: Ref[ItemModel] = itemBuilder.newInstanceRef(itemBuilderStrategy)
```

At this stage, for each trait in the model, the builder holds a trait factory, which may
be asked to create the trait's proxy object.

Then the developer configures the builder strategy, which is used by the builder
during the instantiation to consult which trait composition should be selected for
the instantiation. The `HintClassBuilderStrategy` strategy allows specifying traits that
should be present in the selected trait composition.

The strategy object may actually contain inconsistent hints, but they will never lead
to using an inconsistent composition of traits. Hints only helps the builder
to select the best matching trait composition to the given hints. Without any
hint the class builder selects the first composition from the list of all possible
compositions.

After the builder selects the valid trait composition, after consulting the selection
with the builder strategy's hints, it invokes the corresponding trait factory for
each trait in the selected composition.

The builder can be used repeatedly to instantiate various trait compositions.

```scala
val otherItemRef: Ref[ItemModel] = itemBuilder.newInstanceRef(otherItemBuilderStrategy)
```

Since the builder uses always the same trait factories, it becomes very important,
whether a factory is stateless or stateful. If a factory is stateless it always
creates a new trait proxy instance. On the other hand, if a factory is stateful
it may implement some instance management such as pooling. In the simplest case
it works as a singleton factory, which creates only one instance.

If all trait factories are singleton factories, then the same trait proxy objects
may appear in different trait line-ups produced by the same builder.

The state of the singleton factories before the first instantiation.
<div>
<img src="http://zslajchrt.github.io/resources/unitializedProxies.png" />
</div>

The state of the singleton factories after the first instantiation.
<div>
<img src="http://zslajchrt.github.io/resources/partiallyInitializedProxies.png" />
</div>

The state of the singleton factories after the second instantiation.
<div>
<img src="http://zslajchrt.github.io/resources/fullyInitializedProxies.png" />
</div>

Such a configuration reminds *object morphing*. The instances returned by the builder
with singleton factories only may be seen as different forms of
one *multi-instance* consisting of the trait proxy objects held by the singleton
factories in the builder.

After the item builder is configured, it returns the result wrapped in
a special `Ref` object.

This object, in contrast to the plain reference returned by `newInstance`,
keeps a link to the model, from which the instance is created. Using the `Ref`
object reference instead of the plain reference allows performing additional compile-time
checks when the reference is passed to a method as its argument.

The instance may be retrieved by invoking the `instance` method on the reference object.

```scala
  val item: Thing with Material with Shape = itemRef.instance
```

Now, let us turn our attention to the target domain model. It may be
expressed in terms of a one-dimensional model with two concrete traits
`ScannedBanknote` and `ScannedCoin` belonging to the `ScannedCurrency` dimension.

```scala
type ScannedCurrencyModel = ScannedBanknote or ScannedCoin
```

This model generates only two trivial trait compositions and it may be tempting
to create an instance of this in a similar way as the item instance, as shown in
the previous section.

```scala
val curBuilder = new ClassBuilder[ScannedCurrencyModel]()
... // the strategy configuration
val curRef = curBuilder.newInstanceRef(curBuilderStrategy)
```

Unfortunately, it will not work. The reason is that the traits `ScannedBanknote`
and `ScannedCoin` are dependent on other traits via their self-type. In other
words, the model used in the builder is incomplete.

To make it complete we would have to extend it by the required dependencies:

```scala
type ScannedCurrencyModelComplete =
  (ScannedBanknote with Banknote with Rectangle with Paper) or
  (ScannedCoin with Coin with Cylinder with Metal)
```

Now we could make an instance of this model, however, we would have to copy
data from outer instances of the selected traits. And this is exactly what we
do not want. What we want is to reuse those outer trait instances by the
scanned currency builder.

Reusing the item's trait proxies in the currency builder should be as easy as
passing the item's reference object to the constructor of the currency builder:

```scala
def makeCurrency(itemRef: Ref[ItemModel]): Ref[ScannerCurrencyModel] = {
  val curBuilder = new ClassBuilder[ScannedCurrencyModel](itemRef)
  ...
}
```

Now, the compiler has all necessary information about the the target model (which is incomplete)
and the source model retrieved from the object reference type (i.e. `ItemModel`).
The compiler must be able to determine if the source model can deliver all
trait proxies required by the traits in the incomplete target model.

In this case the compiler should fail, since the source model cannot deliver
`Banknote` and `Coin` trait proxies.

One solution could be to extend again the `ScannedCurrencyModel` by `Banknote`
and `Coin`, but it would lead to another copying of data from external currency
instances returned by the currency database to the instance created by the builder.

```scala
type ScannedCurrencyModelCompleteWithCurrency = (ScannedBanknote with Banknote) or (ScannedCoin with Coin)
```

There is, however, another way to incorporate an existing trait instance
to a new composition: the target model may be extended by special traits
implementing the factories for the missing traits. Since the two missing
traits are `Banknote` and `Coin`, the extended target model will look as follows:

```scala
type ScannedCurrencyModelWithLoaders = (ScannedBanknote with BanknoteLoader) or (ScannedCoin with CoinLoader)
```

The `BanknoteLoader` and `CoinLoader` are trait factories providing the missing
trait proxies and may be implemented as shown in this listing:


```scala
trait BanknoteLoader extends TraitFactory[Banknote] {
  this: Paper with Rectangle =>

  def load(): Option[Banknote] = {
    currencyDb.findBanknote(width, height, color)
  }
}

trait CoinLoader extends TraitFactory[Coin] {
  this: Metal with Cylinder =>

  def load(): Option[Coin] = {
    currencyDb.findCoin(2 * radius, height)
  }
}
```

The `BanknoteLoader` implements the `TraitFactory` interface and specifies its
dependencies on `Paper` and `Rectangle` by the self-type. This self-type gives
the loader access to the paper's and rectangle's properties, which are used to locate the
corresponding banknote object in the currency database in the `load` method.
The result of the method is optional, which allows to indicate a failure to
find the banknote in the database by returning `None`.

The `CoinLoader` works analogously.

Note: A trait factory must not depend on the trait it creates.

Now we can pass the target model with the loaders to the currency builder's
constructor along with the item's reference object.

```scala
def makeCurrency(itemRef: Ref[ItemModel]): Ref[ScannerCurrencyModel] = {
  val curBuilder = new ClassBuilder[ScannedCurrencyModelWithLoaders](itemRef)
  loaderBuilder.newInstanceRef
}
```

The compiler should not complain now, since all missing trait proxies may be
delivered either by `ItemModel` or by the currency loaders.

The target model effectively shrinks the number of possible input compositions
from four to two, because only `Thing with Metal with Cylinder` and
`Thing with Paper with Rectangle` matches with the target model.

```
ScannedBanknote with BanknoteLoader with Banknote -> Thing with Metal with Cylinder
ScannedCoin with CoinLoader with Coin -> Thing with Paper with Rectangle
```

It is important to emphasize, that there is no builder strategy used in this case,
since the resulting composition is determined by the composition wrapped in
`itemRef`. The currency builder in fact inherits the strategy from the `itemRef`.

The effective joined model may be expressed as follows:

```
(Thing with Metal with Cylinder with Coin with CoinLoader with ScannedCoin) or
(Thing with Paper with Rectangle with Banknote with BanknoteLoader with ScannedBanknote)
```

A possible state of singleton factories in the joined model.
<div>
<img src="http://zslajchrt.github.io/resources/joinedProxies.png" />
</div>


There are two scenarios, however, in which the references's `instance` method may fail:

* The item reference carries an unmapped composition (i.e. `Thing with Metal with Rectangle` or `Thing with Paper with Cylinder`)
* A currency loader does not find the corresponding currency record in the database.

Both scenarios are considered application exceptions, i.e. are part of the use-case
and must be handled by the application.

In order to facilitate handling of such conditions, the reference object
provides the `maybeInstance` method, which returns `Some(instance)` in case of success
or None otherwise.

```scala
  val itemOpt: Option[Thing with Material with Shape] = itemRef.maybeInstance
```

The conclusion is that mapping based on the class builder

* may ensure consistency of target compositions at compile-time
* may avoid loosing behavior and data in successive mappings
* may reuse more proxies

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
character by state (e.g. delegation) whereby they incorporate type schizophrenia
into the model.

* The concept of traits as implemented in Scala or Groovy fits very well to
the needs of such a mapping.

* However, using static traits to express multidimensional objects leads to the explosion
of class declarations and the code as such. Using dynamic traits solves this problem.

* Neither static nor dynamic traits solve the problem of copying states between
source and target instances during the mapping.

* The solution may be a mapper based on the concept of the class builder

#####Source Code

The following [link](https://github.com/zslajchrt/morpheus-tutor/tree/master/src/main/scala/org/cloudio/morpheus/scanner)
refers to a source code containing a runnable implementation of the Scaner protodata
case study developed by means [Morpheus](https://github.com/zslajchrt/morpheus).
