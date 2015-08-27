---
layout: post
title: Developing Protean Application With Morpheus - Part 6
comments: true
permalink: developing-protean-applications-part6
---

####Protean Services - Cont.

The previous analysis showed that modeling a protean component without traits
leads inevitably to a design suffering from serious flaws, most of them caused
by object schizophrenia, which results from using delegation instead of inheritance
(or extension).

In this article I will be dealing with modeling the mail service by means of traits.
Scala and Groovy are two representatives of a platform adopting the concept of traits.

####Modelling Protean Service With Traits

The domain model designed by means of traits is at first sight very similar to
that of the non-trait case.

<div>
<img src="http://zslajchrt.github.io/resources/userAccountsTraits.png" width="380" />
</div>

However, there are several substantial differences.

First, the `PremiumUser` type is a real trait, which, in contrast to the interface `PremiumUser`,
extends the concrete class `RegisteredUser`. This extension explicitly declares
that it is meant to be a trait of the registered user. Also, this trait can be applied
on `RegisteredUser` only.

Second, the adapter types are also traits and not concrete classes.

```scala
trait EmployeeAdapter extends MailOwner {
  this: Employee =>

  override def nick(): String = employeeCode

  override def email(): String = employeeCode + "@bigcompany.com"

  override def birthDate(): Date = birth
}
```

Implementing the adapters as traits makes them "composable" with other types in
the application, in particular with the `UserMail` implementations. This is
the important step toward the elimination of the delegation.

The adapter traits do not extend the corresponding account classes as their
counterparts in the no-trait design do. Instead, each adapter specifies its self-type
to refer to the corresponding account class. To put it briefly, the `self` type of
a trait specifies a type with which the trait is supposed to be composed.
Specifically, the `EmployeeAdapter` may only exist in a composition with `Employee`.
The self-type also makes public and protected members accessible from the trait,
actually, it behaves as if the trait privately extended the self-type, i.e.
the trait does not expose the self-type.

As far as the mail service is concerned, it also utilizes traits as much as possible
with the intent to eliminate delegation. All `UserMail` implementations
`DefaultUserMail`, `EmployeeUserMail` and `RegisteredUserMail` are traits. It
makes possible to compose them with the adapters as well as with the account classes.
Each mail service trait also specifies the self-type to express its dependency on other
types.

The composition of the account classes, adapters and mail service types is depicted
on the following diagram.

<div>
<img src="http://zslajchrt.github.io/resources/mailServiceTraits.png" width="450" />
</div>

If we compare this diagram with its no-traits counterpart, there is
a significant difference: there is no delegation. All references are replaced
by extensions. It follows, that there is no loss of type information, since
the resulting mail service is an instance of a class completely composed of all
constituting types. For example, it is possible to invoke `mailService.isInstanceOf[Employee]`
to determine whether the email service is built for an employee account.

Not even the `VirusDetector` introduces any delegation in contrast to the no-trait
design. As expected, the `VirusDetector` is implemented also as a trait.

<div>
<img src="http://zslajchrt.github.io/resources/virusDetector.png" width="180" />
</div>

Since `VirusDetector` is a trait, it is possible to implement the `validateEmail`
method only and omit the overriding of `sendEmail` to invoke explicitly the validation.
This workaround is no longer necessary, since there is no delegation and the trait
becomes one monolith including `DefaultUserMail`. This default email service implementation
invokes `validateEmail` within its `sendEmail` method. The invocation jumps into
the top-most trait overriding the `validateEmail` method, from where the invocation
may be propagated by `super.validateEmail(msg)`.

Similarly to the `VirusDetector`, the `UserFaxByMail` is also implemented as a trait,
as shown in the following picture.

<div>
<img src="http://zslajchrt.github.io/resources/faxServiceTraits.png" width="180" />
</div>

The trait is applied only if the user is a premium customer. This condition is
stressed by the self-type of `UserFaxByMail`.

So far the design has manifested no flaws. Unfortunately, the implementation
of the `AlternatingUserMail` must introduce some delegation. As seen in the
following figure, the model is identical to that of the no-trait case.

<div>
<img src="http://zslajchrt.github.io/resources/alternatingTraits.png" width="320" />
</div>

The reason is that the trait platform has no type-safe tool to implement an instance,
whose composition could be modified in real-time. Thus, all issues described
in the no-trait case hold here as well.

The whole system is depicted on the following diagram:

<div>
<img src="http://zslajchrt.github.io/resources/mailServiceTraitsAll.png" width="620" />
</div>

#####Assembling the Service

The following code sketches how the email service may be assembled from individual
components.

```scala
def initializeMailUser(employee: Employee, registeredUser: RegisteredUser): UserMail = {

  val empMail = new Employee() with
    EmployeeAdapter with
    DefaultUserMail with
    EmployeeUserMail with
    VirusDetector

  empMail.adoptState(employee)

  val ruMail = if (registeredUser.premium)
    new RegisteredUser() with PremiumUser with
      RegisteredUserAdapter with
      DefaultUserMail with
      RegisteredUserMail with
      VirusDetector with
      DefaultFaxByMail
  else
    new RegisteredUser() with
      RegisteredUserAdapter with
      DefaultUserMail with
      RegisteredUserMail with
      VirusDetector

  ruMail.adoptState(registeredUser)

  new AlternatingUserMail(empMail, ruMail)
}
```

The first look on the code suggests that the assembling code is more compact. It is
caused mainly by the Scala's syntax and partly by using traits. For example,
it is not necessary to define the auxiliary `Premium` class, since the `PremiumUser`
trait can be specified as part of the anonymous class signature in the `new` statement.

The complete source code may be viewed [here](https://github.com/zslajchrt/morpheus-tutor/tree/master/src/main/scala/org/cloudio/morpheus/mail/traditional).

####Summary

We still need to clone the state of both preexisting `employee` and `registeredUser`
instances. The adoption methods are annoying.

`EmployeeUserMail` and `RegisteredUserMail` are now more general since they extend
`UserMail` and not `DefaultUserMail`.

`VirusDetector` trait is specified twice; this duplicity may cause several problems:

1. `VirusDetector` contains a virus counter, which will exist in two copies. It may
cause problems when monitoring the counter, for example.
2. The two `VirusDetector` instances may unnecessarily use more system resources
3. When refactoring it is easy to omit some VirusDetector's occurrence

A solution could be to apply the `VirusDetector` as a trait of `AlternatingUserMail`.
Unfortunately, in such a case the virus detector's `validateEmail` method
would not be invoked from `DefaultUserMail.sendEmail` because of delegation.

The complete type information propagates well until the instantiation of the
`AlternatingUserMail`. Therefore the client must be still tightly coupled with
`AlternatingUserMail` and nothing really changes from his perspective.

If only that were possible to overcome this last obstacle. Then we would get
the ideal implementation of the mail service with lossless propagation of types.
