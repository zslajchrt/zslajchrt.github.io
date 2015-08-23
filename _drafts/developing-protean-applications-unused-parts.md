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
