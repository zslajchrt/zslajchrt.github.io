---
layout: post
title: Developing Protean Application With Morpheus - Part 8
comments: true
permalink: developing-protean-applications-part8
---

###Modelling Protean Services With Class Builder (static/dynamic)

In the section dealing with the modeling of multidimensional data [LINK] is shown
that the current languages and platforms do not match perfectly with such a task.
Scala as a representative of a statically typed platform is not able to represent
statically the multitude of combinations arising from the multidimensionality.
On the other hand, Groovy as a dynamic language with some support of static features,
offers a very high flexibility, which, however, cannot guarantee consistency
of trait compositions made at run-time.

It has also been demonstrated that the concept of a class builder, which combines
declarative and dynamic features, may be a solution to problem of modeling
complex multidimensional domains.

The goal of this section is to examine whether the class builder can solve
the main issues associated with the purely dynamic and static approaches.

####Modelling the Domain

We will begin with modeling the data objects, on which the mail service is built on.
The multiform character of the data can be easily express declaratively by the
following type expression.

```scala
type UserModel = Employee or (RegisteredUser with (PremiumUser or Unit))
```

The type may be interpreted as follows: a user may be either an employee or
a registered user, which may also be a premium user.

When rewritten to the equivalent disjunctive normal form, the type reveals
three possible compositions:

```
Employee or
(RegisteredUser with PremiumUser) or
RegisteredUser
```

A user instance is created by means of the class builder, whose constructor
is specified with `UserModel` and the trait initializer. The model allows
the compiler extension to check the consistency of the model. For example, `PremiumUser` requires through
its self-type that it can appear only in the compositions including `RegisteredUser` too.
The trait initializer allows an additional initialization of trait proxies.

```scala
val userBuilder = new ClassBuilder[UserModel]({
    case emp: Employee => emp.initFromMap(employeeData)
    case ru: RegisteredUser => ru.initFromMap(registeredUserData)
})
```

The builder goes hand in hand with the builder strategy when creating a new
instance. The strategy recommends, which compositions or *alternatives* fits
best with the context or input data. A strategy implements `isRecommended`
interface, which includes one method returning a boolean value reflecting
the strategy's opinion on the alternative passed as the argument.

```scala
trait ClassBuilderStrategy {
  def isRecommended(alt: Alternative): Boolean
}
```

The mutating nature of the user, which alternates between being `Employee`
and `RegisteredUser`, requires that we create a special implementation
of `ClassBuilderStrategy`.

```scala
val userBuilderStrategy = new ClassBuilderStrategy {
  def isRecommended(alt: Alternative): Boolean = if (isOfficeHours) {
    alt.containsTrait[Employee]
  } else {
    userData.get("premium") match {
      case true => alt.containsTrait[RegisteredUser] && alt.containsTrait[PremiumUser]
      case _ => alt.containsTrait[RegisteredUser] && !alt.containsTrait[PremiumUser]
    }
  }
}
```

During the office hours this strategy is recommending the alternative containing `Employee`,
otherwise it is recommending the alternatives containing `RegisteredUser` either with or
without `PremiumUser` in accordance with the value of the `premium` property
in the registered user input data.

The user reference is created as follows:

```scala
val userRef = userBuilder.newInstanceRef(userBuilderStrategy)
```

Such a reference object may be passed to other methods for subsequent mappings.

####Modelling the Behavior

Now let us turn our attention to the modeling of the email service. Its interface
consists of a fixed and optional parts. The fixed one provides methods for
sending and validation of emails, while the optional one, which is available
for premium registered users only, allows the user to send faxes by email.

The behavior of the fixed interface has an invariant component, which scans email
attachments for viruses, while the variant component varies according to
the underlying user object. If the user is an employee, the service automatically
appends the employee's signature. If the user is a registered user, the service checks
if the user's license is still valid.

The described behavior can decomposed to traits and modeled by the following
type:

```scala
type MailServiceModel = DefaultUserMail with VirusDetector with
  ((EmployeeUserMail with EmployeeAdapter) or
   (RegisteredUserMail with RegisteredUserAdapter with (DefaultFaxByMail or Unit)))
```

This type can be rewritten to the following disjunctive normal form, which
shows the three alternatives (trait compositions) corresponding to the states
of the email service.

```
(DefaultUserMail with VirusDetector with EmployeeUserMail with EmployeeAdapter) or
(DefaultUserMail with VirusDetector with RegisteredUserMail with RegisteredUserAdapter with DefaultFaxByMail) or
(DefaultUserMail with VirusDetector with RegisteredUserMail with RegisteredUserAdapter)
```

A new reference to the mail service may be constructed by the class builder
by specifying the incomplete `MailServiceModel` as the type parameter of the constructor and
by passing the `userRef`, which will deliver the missing traits.

```scala
val mailServiceBuilder = new ClassBuilder[MailServiceModel](userRef)
val mailServiceRef = mailServiceBuilder.newInstanceRef
```

Let us have a look now, how the service model alternatives actually map to the alternatives
of the user model:

```
DefaultUserMail with VirusDetector with EmployeeUserMail with EmployeeAdapter -> Employee
DefaultUserMail with VirusDetector with RegisteredUserMail with RegisteredUserAdapter with DefaultFaxByMail -> RegisteredUser with PremiumUser
DefaultUserMail with VirusDetector with RegisteredUserMail with RegisteredUserAdapter -> (RegisteredUser or RegisteredUser with PremiumUser)
```

The first mail service alternative maps exactly to one alternative of the user model,
because `EmployeeAdapter` depends on `Employee`. Similarly, the second alternative
maps exactly to one user model alternative, since only one has both `RegisteredUser`
and `PremiumUser` as required by `RegisteredUserAdapter` and `DefaultFaxByMail`.

However, the third alternative depending on `RegisteredUser` through `RegisteredUserAdapter`,
can be satisfied by two user model alternatives. This ambiguity may cause problems
when the class builder instantiates a mail service and is selecting the target alternative
for a user instance implementing both `RegisteredUser` and `PremiumUser` traits,
since there are two eligible alternatives in the mail service model.

This problem may be solved by taking into account the sorting order of the target
alternatives: the first matching target alternative wins. So in this case the second
alternative (with `DefaultFaxByMail`) would win.

####Runtime Morphing

For the client of the mail service, such as the UI, the mail service mutations, which
take place in the background, should be as transparent as possible. The core
functionality should always be available regardless of the current composition
of the service. Thus, the client must be given a mail service reference, which
handles such changes quietly and is usable all the time the service is offered.

Such a reference may be obtained from the reference wrapper object through
the `mutableInstance` method.

```scala
val mailService: UserMail = mailServiceRef.mutableInstance
```

The returned object is in fact a proxy holding a reference to the current mail
service composition of traits. The proxy implements the lowest upper bound type (LUB) for
all target alternatives by delegating on this reference. (The LUB in this example
is `UserMail`.)

To refresh the proxy's delegate reference could be a responsibility of the proxy itself
or of an external agent other than the client.

Besides the LUB, the proxy also implements the "control" interface `MutableInstance`,
which allows the client to control the proxy:

```scala
val mailService: UserMail with MutableInstance[MailServiceModel] = mailServiceRef.mutableInstance
```

The control interface exposes the reference through the `delegate` method.
This method may be used in situations, when the client wishes to customize
its appearance, for example.

To determine for which type of user the email service is currently provided, one
may call `delegate` and check the result type by `isInstanceOf` or by the `match` block:

```scala
mailService.delegate match {
  case Employee => // handle an employee
  case RegisteredUser with PremiumUser => // handle a premium registered user
  case RegisteredUser => // handle a registered non-premium user
}
```

In a similar manner the client can detect that the service currently implements
the `FaxByMail` extension:

```scala
mailService.delegate match {
  case fax: FaxByMail => // use fax
  case _ =>
}
```

####Summary

* All forms, which a particular object can assume during its lifetime, may be
expressed declaratively by a special type expression. The same expression is used
in the previous case study to describe all forms, with which object can be born with.
Such an object keeps the form for all its life. Thus, the concept of the class builder
developed in this section generalizes the concept from the previous case study.

* The type expression defines the object morphing model and is analogous to
the finite state machine.

* All advantages described in the previous case study [LINK] hold here as well,
such as the consistency check at compile-time, preservation of types and behavior
during mappings, no combinatorial explosion, no need for copying, wrapping and delegating.

* The only delegation is present in the mutable proxy, which is, however, handled
on the platform level, and not on the application level. The instance held by
the delegating reference in the proxy never sinks as in the case of the repeated
wrapping in the previous approaches.

* It is possible to avoid creating redundant instances of the same trait as in the case of
`VirusDetector` in the previous approaches.

* The client may check the availability of extension interfaces (e.g. `FaxByMail`)
through the `delegate` method on the mutable instance; `AlternatingUserMail` is no longer needed.

* All trait proxy objects, regardless of many mappings the underwent, remain on
the top of the instance, i.e. they are not "sinking" after every mapping.
