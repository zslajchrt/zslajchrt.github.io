---
layout: post
title: Developing Protean Application With Morpheus - Unused Parts
comments: true
permalink: developing-protean-applications-unused
---
###So What Protean Means?

The word actually refers to Proteus, a god of sea and rivers in Greek mythology,
also called the god of "elusive sea change". [The Free Dictonary](http://www.thefreedictionary.com/protean)
defines *protean* as follows: *Readily taking on varied shapes, forms, or meanings*. And this
definition fits exactly the character of both data and behavior found in many
today's big-data applications.

###Protean Data

Today, as the amount of available information and its sources is growing almost
uncontrollably, it becomes increasingly difficult to manage it by traditional means
and practices.

Besides its abundance, the heterogeneity of data is a key factor in the struggle to
harness *big data*. We often have to keep the data as it is, i.e. heterogeneous,
since any normalization would lead to some loss of information, for instance.

TODO: The entity may be scattered across various databases ....

In extreme case a data entity can be so *protean* that the only attribute, which is
always present, is the entity's identity, while all other attributes or
groups of attributes are more or less arbitrary or specific to a group of entities only.

In such a case it does not make much sense to try to describe such a data
structure by means of any schema, no matter how bushy the schema would be.
Such an entity is naturally resistant to an all-embracing description, i.e. *metadata*.

Previously, metadata usually came into existence in the framework of a detailed data
analysis process, i.e. its existence **preceded** that of the data. The resulting metadata
was subject to a very careful change management and the change rate tended to be very low
because everybody was afraid of it. To wrap up, **the metadata precedes the data**.

On the other hand, when collecting large amounts of heterogeneous data, sometimes we have
a very limited knowledge of the data in the beginning. We may call such data
collected in the early stages as *protodata*, since it has a very limited applicability
and only with the arrival of more data we can expect, or hope at least, that some
metadata will emerge.

We may conclude that in this situation **the data precedes the metadata**.
Furthermore, there may emerge other alternative metadata describing the same data.

The metadata in this concept can be seen as a pattern or a template used to select groups of
associated entities sharing some qualities rather than a rule dictating
the structure of the data. These associations of entities establish *domains* within
which their data yields a specific kind of information. The metadata also establishes
relationships between entities in the domain forming so a network of friends,
colleagues or beer lovers, for instance.

Above that, the same entity may be present in different domains, thus, its meaning
depends on the interpretation of the subject that understands the domain.

From what has been described above we can see that such data varies in three
aspects making it *protean*: shapes/content, forms/format, meanings/domain-specific
or subjective interpretation (metadata).

*Note: Relational tables can be viewed as a special kind of domain templates.
It is as if a relational table were just a view to a set of entities in our protodata,
which follow the rules given by the table schema. In theory we could build a universal
RDBMS system working within various domains identified in a NoSQL database. Leaving
aside possible practical problems with performance, distribution etc., of course.*

###Protean Behavior

Having started storing extremely protean data it is still too early to begin to develop
some meaningful application. Not to mention to develop something what had emerged in
the heads of analysts long before we actually started receiving the stream of data.
Such an approach would clearly lead to a waste of time and resources. The typical
reasoning at that time is: *We believe that our huge data must be full of gems.
The problem is that we don't know these gems nor how to retrieve them.*

It does not mean that we should not muse about potential uses, but we should
be prepared to abandon them quickly and change the course when new insights,
dug from the data, reach us.

What we should do, however, is to examine and wring the protodata as much as
possible with the goal to discern any glimpse of some "alliances" of entities that
could be described by some schema and be given some meaning. Such a thing can be
a fetus of a new insight, which can drive the further development of an application
utilizing the insight.

Once a well-defined domain is available (i.e. metadata), we can build
an application for it. As indicated above, domains consist of a network of
interconnected entities, which follow the domain rules anchored in the metadata.

To conclude, we do not build applications directly on top of protodata.
Instead, firstly we identify domains, i.e. subsets of data entities having
something in common, then we describe it by means of metadata giving it some
meaning and establishing relationships between the entities (network).
The resulting domain is the foundation on top of which we can build some applications.

###Technologies

The tools that we need to build protean systems and applications can be grouped
to four groups reflecting the stages of the development.

  1. Persistence
  2. Analytics tools to identify domains
  3. Domain management, network builders
  4. Development tools

####Persistence

There is a number of recently developed storage systems like NoSQL databases,
which can handle peta-bytes of heterogeneous data, however, often at the expense
of the relationships (no joins) and metadata (no schema) management, among others.

It means, thus, that these missing tasks must be solved somewhere in the stack of
application layers. And the lower such a management takes place the better.
Ideally, it would be the language, or eventually the language platform, which
would take on this responsibility. In the worst case, it would be done by the application
itself. Usually it is done by an intermediary big data platform.

####Analytics tools

The big-data analytics has advanced pretty far over the past decade. Technologies
like Hadoop, Spark, Impala, Splunk and many others may be used to identify domains in
protodata.

The output of the domain analysis is a description of the new domain. The description
should allow a straightforward introduction of the new domain into the system.

This task may require an experienced data science team, which is supposed to uncover
numerous categories of complex dynamics of interactions among entities from noisy protodata.

####Domain management

The domain management basically solves these tasks:

  1. Introducing new domains (metadata) into the system
  2. Tracking entities in protodata and assigning them to existing domains (online or offline tagging)
  3. Indexing
  4. Building networks on top of the entities in a domain (relationship management)
  5. Mediating the communication between applications and domain (connection management, queries etc.)

These tasks may remind a database engine, which de-facto is in a broader sense of the term.
At the time of writing this article I am not aware of any technology that would
address these requirements optimally.

The domain management should avoid any shuffling or copying of data. There is
always only one instance of a data entity in the protodata, even if
the entity may belong to several domains. These domains may represent the
entity in different shapes so as to make its usage easier for domain applications,
however, under the hood, it is always the same entity instance playing various
roles only.

The domain also *animates* its entities by attaching characteristic behavior to them.
For instance, if the domain consists of pictures, every picture entity would
be enhanced by methods for editing, such as resizing, rotating, applying various effect
filters etc. It is important to emphasize, that this enhancement is restricted to the scope of
the picture domain only. So if an entity belonging to domain `A` belongs also
to domain B, the enhancements of the entity made in `A` are not seen in `B`
and vice versa. And of course, this does not guarantee that changes of the entity's state
made in `A` through its enhancement are not seen in `B`.


####Development tools

Developing applications on top of domains should be much easier than doing
the same thing on top of raw protodata. The reason is that domains served
by the domain management contain a lot of functionality that would have to be
implemented by the application otherwise. Also the domains should be as much
clearly defined as possible.

As we will see in the next part of this article, even in a simple application
built on top a simple network the same entity may assume various roles in dependence
on the other role with which it has the relationship.

Dynamic languages like Python, Ruby or JavaScript may sound like good candidates,
however, the refactoring of protean application written in these languages would become
extremely error-prone. It is pretty easy to forget to change, add or remove
something somewhere while something new is being added to the application since
the network of mutual relationships between entities can be very complicated.

##What's Next?
In the next part I will not be dealing with the technology stack
needed for the persistence of protodata nor with analytics tools for domain identification.
Instead I am focusing on building applications on top of a domain.

The goal is to show how with [Morpheus](https://github.com/zslajchrt/morpheus) and Scala
we can build protean applications while preserving all benefits associated with Scala's strong
static type system along with additional **type-safe metamorphism** of objects
provided by Morpheus.


```
              Shape
    Circle   Rectangle  Other
```

```
             Material
    Plastic      Paper     Metal
```

```
            Polymer
  PVC  Polystyrene Polypropylene ...
```

```
        Plastics Classification
  Thermoset   Thermoplastic   Elastomers
```

 without having some intention of using them.

The contents of a wallet - banknotes, coins, vouchers, paychecks, receipts, tickets, business cards, credit cards, notes.
What can they be used for?

I need to pay for something: {banknotes, coins, vouchers, paychecks, credit cards}
Evaluated attributes: {value/balance, currency}
Possible targets: {whole range of goods, services, fines etc.}

I need to make a quick note on something: {vouchers, banknotes, receipts, tickets, notes}.
Evaluated attributes: {empty white space, surface}
Possible targets: {a phone number, a rhyme, an idea}

I need to open something: {coins, credit cards, business cards}
Evaluated attributes: {thickness, hardness}
Possible targets: {mobile, bolt, can ...}

I need to find a phone number: {receipts, business cards, notes, tickets}
Evaluated attributes: {phone, label}
Possible targets: {taxi, friend, theatre, office, shop, police ...}

The actual behavior in each scenario depends on objects in the wallet as well as
on the concrete purpose, i.e. the target. For instance, if I am going to pay for a cup of coffee (the target)
I will use rather coins or smaller banknotes, if I have any, than big banknotes or paychecks.

In each example there are two sources of behavioral mutability: the diversity of the wallet
objects and that of the targets.

Of course, there may exist scenarios with more sources of mutability
such as the context of the scenario (I will not use the coins to pay for the coffee
since I have to use them to purchase a tram ticket at a ticket machine).

There is no apparent hierarchy (tree). The hierarchy emerges only after we know
for what we want to use it.

data instances (objects) differ on the syntax (physical) level (data level), however,
they carry the same kind of information.

The same protean data can carry more kinds of information.

carrying the same information => ability to play a role in an interaction

####Commonalities vs. Distinctions

All objects eligible for a scenario must be able to yield the same kind
of information required by the scenario. All objects may provide such information
through different attributes whose values may be encoded in distinct formats.

The shared information amount is indirectly proportional to the number of
eligible object types, the so-called **domain size**. The bigger the domain
size, the less shared information, however the better applicability of
the **default behavior**.

The default behavior handles all eligible objects in the same way and ignores
their specifics, i.e. it works only with the **shared attributes**.

Further, the amount of the shared information among the objects positively
influences the **fitness** of the default behavior of the scenario. The smaller
the domain, the more shared information and the fitter default behavior.
Unfortunately, it results in worse applicability.

Obviously, banknotes and coins have more in common than banknotes, coins and credit cards.
The default payment procedure for banknotes and coins only can theoretically take
into account the physical properties of the currency in order to organize
the weight and space in the wallet. Such properties make no sense for
credit cards, since they remain in the wallet after the payment.

It may happen, however, that the customer will not be able to pay for the goods since
his 'perfect' space-optimizing payment default behavior does not know how to use
a credit card.

Since the to maximize the applicability of the

composition, adaptive behavior

The less the domain size,
the bigger the information share, the fewer exceptions and the fitter the default behavior.

Conclusion: The fitness of the default behavior and the domain size of a scenario
are contradictory qualities.

sharing the same information = commonality

"stupid" behavior

the behavior should reflect the actual objects


###Identifying Users

The GigaMail service is only available for those who are registered or are
employee of the Big Company.

A user visiting the service is requested to sign in or register. An employee
is not required to register since his or her profile is retrieved from the
company's internal database.

After signed in, the user is associated with his or her profile, which can
have two forms: the registered user's profile or the employee's profile.

```json
{
  "public": {
    "nick": "joe4",
    "firstName": "Joe4",
    "lastName": "Doe4",
    "email": "joe4@gmail.com",
    "male": true,
    "birthDate": "1971-02-21T00:00:00Z"
  },
  "license": {
    "premium": true,
    "validFrom": "2015-02-21T00:00:00Z",
    "validTo": "2016-02-21T00:00:00Z"
  }
}
```

This is an employee's record from the Big Company's internal database.

```json
{
  "employee": {
      "employeeCode": "xyz9000",
      "position": "Developer",
      "department": "R&D",
      "personalData": {
        "firstName": "Joe1",
        "middleName": "D.",
        "lastName": "Doe1",
        "title": "Mr."
      }
    }
}
```

###Modeling User Profiles

```scala
case class RegisteredUserPublic(nick: String, firstName: String, lastName: String, email: Option[String], male: Option[Boolean], birthDate: Option[Date])

case class RegisteredUserLicense(premium: Boolean, validFrom: Date, validTo: Date)

case class RegisteredUser(`public`: RegisteredUserPublic, license: Option[RegisteredUserLicense])
```

```scala
case class EmployeePersonalData(firstName: String, middleName: Option[String], lastName: String, title: String, isMale: Boolean, birth: Date)

case class Employee(employeeCode: String, position: String, department: String, personalData: EmployeePersonalData)
```

```scala
@fragment
trait RegisteredUserEntity {

  protected var regUser: RegisteredUser = _

  def registeredUser = regUser
}

@fragment
trait EmployeeEntity {

  protected var emp: Employee = _

  def employee = emp
}
```

```scala
val regUserKernel = singleton[RegisteredUserEntity]
println(regUserKernel.~.registeredUser) // prints null

val empKernel = singleton[EmployeeEntity]
println(empKernel.~.employee) // prints null
```

####Loading Profiles

```scala
@dimension
trait UserDatasources {

  def registeredUserJson: JValue

  def employeeJson: JValue

  def initSources(userId: String)
}
```

```scala
import org.morpheus.FragmentValidator._

@fragment
trait RegisteredUserLoader {
  this: UserDatasources with RegisteredUserEntity =>

  def load: ValidationResult[RegisteredUserEntity] = {
    implicit val formats = DefaultFormats
    this.registeredUserJson.extractOpt[RegisteredUser] match {
      case Some(ru) =>
        this.regUser = ru
        success[RegisteredUserEntity]
      case None =>
        failure[RegisteredUserEntity]("invalid content")
    }
  }
}
```

```scala
val regUserLoaderRef: &[$[RegisteredUserLoader with UserDatasourcesMock]] = regUserKernel
val regUserLoaderKernel = *(regUserLoaderRef, single[RegisteredUserLoader], single[UserDatasourcesMock])
```

TODO: include the link to the source code of UserDatasourcesMock

```scala
regUserLoaderKernel.~.initSources("4")
regUserLoaderKernel.~.load match {
  case Failure(_, reason) =>
    sys.error(s"Cannot load registered user: $reason")
  case Success(_) =>
    println(regUserKernel.~.registeredUser)

}
```

```scala
val empKernel = singleton[EmployeeEntity]
val empLoaderRef: &[$[EmployeeLoader with UserDatasourcesMock]] = empKernel
// analogous to the previous code
```


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
