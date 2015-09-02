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

* If the registered user owns a premium license and this license is still valid,
then the resulting mail service instance is marked by the `Premium` trait and
extended by `DefaultFaxByMail`.
* If the user is mail, the instance is further marked by `Male` or `Female` traits.
* If the use is older then the instance is marked by `Adult`.

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
      }

      return regUserMail;
  }
```

The above-mentioned rules represent three independent dimensions constituting a
configuration space, where every point corresponds to one possible composition
of the resulting mail service instance for the registered user.

The very important fact is, however, that the assembling code no longer blows up
exponentially with the number of dimension. This fact is a consequence of the
step-by-step extensions.

Nevertheless, there are several downsides. First, a new composite instance is
created again and again on each invocation of `getDelegate`, even when the structure
of the new instance remains the same. This problem can be mitigated by storing
the information about the structure of the last instance into a field. A new
instance would be created only if its structure differs from that of the last one.

Second, each extension step leads to the creation of a new proxy object wrapping the
object from the previous step. Here, we can end up in having a cascade of
four proxies (three dimensions plus the proxied `RegisteredUser` entity). It follows
that to get a value of a property in the `RegisteredUser` entity amounts to four
delegated invocations on the nested proxies.

Third, it makes no sense for the extending traits to be stateful, because
the state of an extension attached to the last instance gets lost on every
re-instantiation. Here, it concerns the `VirusDetector` trait, since it holds
a counter of detected viruses in email attachments, which would be periodically
reset. A solution, or rather a workaround, could be to hold the extension's
state out of the extension, which is not a good design practice violating
encapsulation of data.

###Summary

There is no doubt that using dynamic traits helped resolve some problems.

* Combinatorial explosion is over, the number of initialization statements (O(n))
is proportional to the number of dimensions (3) and not to the cardinality
of the cartesian product of the dimensions (O(2^n)).

* We no longer have to clone the state of domain entity objects to the new
extended instances.

On the other hand, dynamic traits did not help solve the problem of metamorphism,
and the client is still tightly coupled with `AlternatingUserMail` class, which
provides the client with an interface to determine the current functionality of
the mail service instance, for example. The metamorphism must still be implemented
by delegation.

Dynamic traits also bring a couple of new problems, some are inherently connected
to the concept of dynamic traits, while others to Groovy's implementation.

* A trait does not have to implement all methods from the extended type, because
it may be assumed that the missing methods will be delivered by other traits when
extending an object. Since the extension mechanism includes no trait completeness check,
it may happen that some methods will be missing after extending the object by
such a set of traits. The dynamic nature of a dynamic traits language also make
it difficult to introduce some reliable completeness check.

* A traits may also depend on other traits. In dynamic languages it is in general also
very difficult to check dependencies at the moment of extending an object, since
the object may or may not contain the required traits. Composing more complex
structures by dynamic traits is therefore prone to missing dependencies.

* Objects can be extended by trait type, not by trait instance. It has important consequences
especially for stateful traits, since their state is initialized at the moment
of extending an object. It follows that while the extended object remains the same
during a series of repeated extensions, the attached traits themselves are always
newly instantiated. It would be useful if we could extend two objects by the same
trait instance to share the trait's state between the two objects.

* A cascade of step-by-step extensions leads to stacked proxies. As a result of it
an invocation of a method belonging to the most bottom trait is mediated by
the whole stack of proxies. (requirement: to flatten the proxies)

* Dynamic languages have a weak type system, which does not allow constructing
composite types, such as `RegisteredUser & Male & Premium & Adult`. Such types
would serve as a sort of type query language used to determine concrete traits
in multiple dimensions:

```groovy
if (userMail instanceof RegisteredUser & Male & Premium & Adult) {
    //...
}
```

In Scala we could use the `match` construct to distinguish between
various combinations of traits:

```scala
userMail match {
  case u: RegisteredUser with Male with Premium with Adult =>
    //...
}
```


```
UserMail with Employee empMail = employee.withTraits(EmployeeAdapter, DefaultUserMail)
```

The above-mentioned findings indicate that because of the lack of a strong static
type system, modeling multidimensional domains by dynamic traits with complex
dependencies is prone to errors and the resulting implementation will tend to end up
in a unmaintainable chaos. With the growing number of dimensions and concrete traits
there will be an increasing probability that some method or some dependency is missing
in the final composition of traits used to extend an object.

The step-by-step concept of extending objects seems to be too unconstrained
delegating the responsibility for the consistency of trait compositions on the developer.
However, considering that the number of trait combinations grows exponentially with
the number of dimensions, the system will quickly run out of control of any developer.

It leads us to the conclusion that only a platform equipped with a strong
static type system allowing for a safe dynamic composition of traits is suitable
for developing complex multidimensional mutable applications must be.
