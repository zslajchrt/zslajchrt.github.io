---
layout: post
title: Developing Protean Application With Morpheus - Unused Parts
comments: true
permalink: developing-protean-applications-unused
---


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
