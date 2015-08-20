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
including its context. [1-WIKI])

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
the proto-domain to the context domain can be extremely difficult.

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
directly map the context domain onto the proto-domain, regardless of the diverse character
of the two domains, and avoid any intermediary processing.

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

/// Figure 2: 2-dimensional diagram

Modeling in multidimensional objects in languages, which do not include traits
or similar concepts, may become rather difficult and the resulting model may not
appropriately grasp the reality. For example in Java, which has no equivalent
to traits, we would have to substitute traits with interfaces and delegation.

/// Figure 3: No-traits diagram

Considering this model, a new item can be created as follows.

```java
  Item item = new Item(10.1, 20.4, 3.1, new Metal(), new Cylinder(0.3, 10.32))
```

However, this substitution comes with some serious design flaws. According to
the model, every item is an instance of `Thing` and two types `Shape`
and `Metal`. In Java, if one needs to know what an object is, (s)he uses
the `instanceof` operator. For example, if an instance of `Item` is tested
whether it is `Thing` or `Material` by this operator, the result will always be true.
On the other hand, testing the item for being `Baggage` will never be true.

```java
  boolean isThing = item instaceof Thing       // always true
  boolean isMaterial = item instaceof Material // always true
  boolean isBaggage = item instaceof Baggage   // always false
```

A problem arises when one wishes to check whether the item is metal. The following
statement will always be false regardless of the possibility that the item may
be, or rather represent, a metal item.

```java
  val isMetal = item instaceof Metal // always false, even if the item is metal
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

On the other hand, let us see how the model looks if the language offers traits.
In contrast to the no-trait model, here the `Thing` type and the two dimensions
are separated. The composition is deferred until the moment of the creation of a new
item.

/// Figure 4: Traits diagram

The following code shows how such a creation with the deferred composition can
look like.

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

```scala
  val isMetal = item.isInstaceOf[Metal]         // true
  val isCylinder = item.isInstaceOf[Cylinder]   // true
```

Now, it is possible to determine the exact type, i.e. what the object exactly is. The reason
is that the identity is no longer scattered among more instances but the item only.

Furthermore, one can use composite types to test instances like in the following code:

```scala
  val isMetal = item.isInstaceOf[Metal with Cylinder] // true or false
```



The absence of traits makes the mapping based on types (nay type-safe) almost impossible.

TODO: Discuss the problem of object schizophrenia

The new context description and the new datasource

/// Figure 5: The context domain diagram

The context domain overview

Mapping the context domain on the proto-domain

/// Figure 6: Mapping context domain to proto-domain diagram

////////////////////////////////////////////////////////

Traits... fine-tuning

Typically, the owner of protodata has currently no use of it.

The God Must Be Crazy

intrinsic properties: shape, material, size, weight ...

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
