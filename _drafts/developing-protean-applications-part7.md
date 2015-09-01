---
layout: post
title: Developing Protean Application With Morpheus - Part 7
comments: true
permalink: developing-protean-applications-part7
---

###Modelling Protean Services With Dynamic Traits

####Introduction

In programming languages with dynamic traits it is possible to add one or more traits
to an object at run-time. A trait is added to an object by invoking a special method on the object and
specifying the trait's type. This provide us with a way to hook
various traits on an object step-by-step, in contrast to the static traits, where the traits
are specified declaratively at once in the initialization statement. The imperative
decoration by traits sounds promising as far as tackling with the problem of
combinatorial explosion of class declarations is concerned.

There are a couple of languages providing dynamic traits, although they offer them
under different names. Among others I have to mention Ruby (modules), Groovy (mixins, traits)
and Python (mixins) [LINKS]. Such languages are usually dynamic with dynamic types.
For the purpose of this section I use Groovy as a representative of such languages.
The main reason is that Groovy is capable of some static type checking and that its
trait syntax is very similar to that of Scala, which I used when dealing with static traits.
In addition, Groovy's syntax is generally very close to the syntax of Java, which
is used here as a non-trait language.

####Modeling the Domain

Groovy traits may be stateful, what makes them a possible replacement for classes
in some cases. For example, when modeling a domain entity we can use a trait
instead of a class. This possibility is especially important when extensions must
not hide extended types. When extending by a trait Groovy actually creates a proxy
wrapping the object and implementing all *interfaces* of the object plus the newly
added traits, which, as opposed to classes, are also considered interfaces.
It follows that if a domain entity were modeled as a class the new proxy object
would loose the information about the entity type. Since the entity can be
modeled by a trait, it is possible to overcome that obstacle.

```groovy
trait Employee {

    private String firstName;
...
}
```

In order to instantiate an entity instance, we have to attach it to an "empty"
object first, since a trait cannot be instantiated alone in Groovy.

```groovy
def employee = new Object() as Employee;
employee.load(employeeData)
def regUser = new Object() as RegisteredUser;
regUser.load(regUserData)
```

The `as` keyword actually creates a proxy object, which wraps the "empty" object
and implements `Employee`, resp. `RegisteredUser`.

*Note: Although Groovy is primarily a dynamic language, it also supports static
compilation. Types or other parts of code annotated by `@CompileStatic`
are compiled statically. [LINK]*

####Implementing the Behavior

Now, when the domain objects are created in a way, which allows loss-less
extension, we can implement the functionality of the mail service.

The diagrams are practically identical to those for the static traits case. The
only main difference is that the types specified in the self-type are added to
the list of implemented interfaces. It might sound confusing, however, the self-type
in Scala is essentially a syntactic sugar for the same thing.

```groovy
@CompileStatic
trait DefaultUserMail implements UserMail, MailOwner {
...
```

As mentioned above, the step-by-step extensions of objects sounds like
a solution to the exponential explosion of statements. In order to make
it more obvious, we will introduce additional dimensions such as
`Gender` with "values" `Male` and `Female`, and `AgeGroup` with values `Child`
and `Adult`.

Since the logic in the traits is the same as in the static traits case, we
will focus on the `AlternatingUserMail` class, whose instance is handed over
to the email client.

The abstract class `AlternatingUserMail` does not wrap any mail service explicitly,
instead, it invokes the abstract `getDelegate` method to obtain the actual delegatee.
In this case, there are basically two mail service instances, one for each
account type. The employee's mail service is used during the office hours, i.e.
8am-17pm, and the registered user's one in the remaining time. The following
listing shows the implementation of `getDelegate` the anonymous class extending
`AlternatingUserMail`.

```groovy
AlternatingUserMail userMail = new AlternatingUserMail() {

    @Override
    UserMail getDelegate() {
      Calendar c = Calendar.getInstance();
      def h = c.get(Calendar.HOUR_OF_DAY);
      if (!(h >= 8 && h < 17)) {
          return getEmployeeMail();
      } else {
          return getRegUserMail();
      }
    }

    ...
```

The `getEmployeeMail` method is trivial; it always returns
the same instance, since the composition of the employee's mail service
is fixed. Its instance is held in `empMail` field in the anonymous implementation
of `AlternatingUserMail`.

*Note: In Groovy, one uses `withTraits` to add traits at run-time to an object.
This method accepts one or more trait types and is invoked on the target object.*

```groovy
    UserMail empMail = employee.withTraits(EmployeeAdapter, DefaultUserMail, VirusDetector)

    UserMail getEmployeeMail() {
        return this.empMail;
    }
```

The `getRegUserMail` method is more complex, since the actual composition of the service
depends on several circumstances. The rules for the composition can be summarized
as follows:
* If the user is 

```groovy
  UserMail getRegUserMail() {
      UserMail regUserMail = regUser.withTraits(RegisteredUserAdapter, DefaultUserMail, VirusDetector);

      def calendar = Calendar.getInstance()

      // Dimension 1
      if (regUser.premium && regUser.validTo.toCalendar().after(calendar)) {
          regUserMail = regUserMail.withTraits(Premium, DefaultFaxByMail);
      }

      // Dimension 2
      if (regUser.isMale()) {
          regUserMail = regUserMail.withTraits(Male);
      } else {
          regUserMail = regUserMail.withTraits(Female);
      }

      // Dimension 3
      def isAdult = (calendar.get(Calendar.YEAR) - regUser.birthDate.toCalendar().get(Calendar.YEAR)) > 18;
      if (isAdult) {
          regUserMail = regUserMail.withTraits(Adult);
      } else {
        regUserMail = regUserMail.withTraits(Child);
      }

      return regUserMail;
  }
```



###Summary

- no cloning

- combinatorial explosion is over, the number of initialization statements (O(n))
is proportional to the number of dimensions (3) and not to the cardinality
of the cartesian product of the dimensions (O(2^n)).

- a downside is that the instances of the services are created over and over.
Also, there may be some stateful traits, whose state would get lost (e.g. `VirusDetector`).

- weak type system, cannot use composite types for variables:
e.g. `RegisteredUser & Male & Premium & Adult`, it is very important for multidimensional
metamorphism, since it provides a sort of type query language.

```
UserMail with Employee empMail = employee.withTraits(EmployeeAdapter, DefaultUserMail)
```

- no compile type check of the assignment

- no shared traits instances, it would be nice if we could extends the employee and regUser by
 the same VirusDetector instance. There remains the duplicity when counting the detected viruses.

- no completeness check, i.e. all methods are implemented by some fragment
- no fragment dependencies check, neither compile-time nor run-time

```groovy
VirusDetector virusDetect = new Object() as VirusDetector;
UserMail employeeAdapt = employee.withTraits(EmployeeAdapter, DefaultUserMail, virusDetect)
UserMail regUserAdapt = employee.withTraits(RegisteredUserAdapter, DefaultUserMail, virusDetect)
```
- the metamorphism must be implemented by delegation, thus the client is still
tightly coupled with a concrete .

If only that were possible to overcome this last obstacle. Then we would get
the ideal implementation of the mail service with lossless propagation of types.

###Wrapping Up

####Type Preservation
Why?

```groovy
if (userMail instanceof RegisteredUser & Male & Premium & Adult) {
    //...
}
```

```scala
userMail match {
  case u: RegisteredUser with Male with Premium with Adult =>
    //...
}
```

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
