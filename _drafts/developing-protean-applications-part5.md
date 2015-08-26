---
layout: post
title: Developing Protean Application With Morpheus - Part 5
comments: true
permalink: developing-protean-applications-part5
---

###Protean Services

In contrast to the previous case study, this one focuses on behavioral
aspects of the adaptive applications development (i.e. protean applications).

Before delving into the analysis, let me introduce the subject by the
following narrative.

*A company plans to offer an email service for both its customers and
its employees. Although the functionality of the service is mostly the same for both types
of users, such as a virus scan, it differs in a couple of details, such as an insertion of
the employees's signature into the end of every email and a verification that
the customer's account is not expired.*

*For eligible users the email service may provide some extending functionality,
such as sending fax by email for premium customers.*

*A number of employees are also company's customers. Such employees are allowed
to use the service under their employee account when they are in the office,
otherwise they use the service under their customer account. To allow a continuous
usage for such users, the email service is able to switch transparently between
the accounts either automatically or manually.*

The description suggests that the service's behavior is very contextual. How it
actually works depends on several factors, such as the type of the current account (employee, customer),
the user's traits (e.g. premium), the message being sent and also time, since
the service can switch between the accounts in the background.

Additionally, the service's interface is also mutable, since at various times
it can provide various extending features. It puts some special requirements on
the service's user interface, which must be able to reflect these mutations.
For example, if a user is both an employee and a premium customer, then the
user interface should adapt its look when the service switches the account to the
premium customer one in the background so that the user could send faxes by email.

In the following paragraphs I am examining how such a service can be
designed on two platforms: the first having no concept of traits and the other
with traits.

####Modelling the Service Without Traits

Before modeling the service itself, let us begin with the domain model. It consists
of two domain entities only: `RegisteredUser` and `Employee`. Since the two
entity classes have no common ancestor, it is necessary to add an adapter interface,
which will establish the common basis for the service, as shown in the following
diagram.

<div>
<img src="http://zslajchrt.github.io/resources/userAccounts.png" width="280" />
</div>

The `MailOwner` interface contains properties having their counterparts in
both entity classes. There are also two implementations of `MailOwner`, each for
one entity class.

Now, when the domain entities can be adapted to a common base, it is possible
to model the mail service.

The `UserMail` interface declares two methods. Method `sendEmail` is the core
method for sending emails. The other method, `validateEmail`, allows the user
to validate a message before sending it.

<div>
<img src="http://zslajchrt.github.io/resources/mailService.png" width="280" />
</div>

The default implementation `DefaultUserMail` works in the context of the current
user account, from which it retrieves the `from` field for emails, among others.
This user account is held in a reference attribute of `DefaultUserMail`.

The `sendEmail` method in `DefaultUserMail` invokes the `validateEmail` method
before the email is actually sent. It allows for subclasses to override the default
validation by adding more specific checks.

As mentioned above, the service's behavior reflects the type of the user account.
Therefore there are two specializations of `DefaultUserMail` for both account types.

`EmployeeUserMail` overrides method `sendEmail` so as to add the employee's
signature at the end of every email.

On the contrary, `RegisteredUserMail` overrides `validateEmail` in order to
check if the user's account is not expired.


The service also contains a stack of pluggable email processors forming
a processing pipeline. These processors implement `UserMail` interface and
are connected through delegation. One of such processors is `VirusDetector`.

<div>
<img src="http://zslajchrt.github.io/resources/virusDetector.png" width="280" />
</div>

Here, one important weakness related to the absence of traits is evident.
The `VirusDetector` class overrides `sendEmail` as well as `validateEmail`,
although it would make sense to override `validateEmail` only. The virus scan
in email attachments may be considered part of the validation performed before sending
the mail by `DefaultUserMail`.

Unfortunately, since the virus detector and the wrapped email service are
separate objects connected by a reference only, the invocation of `validateEmail`
in `DefaultUserMail` will always either remain in `DefaultUserMail` or jump to some
of its subclasses overriding `validateEmail`. Since `VirusDetector` is not a subclass
of `DefaultUserMail`, its implementation of `validateEmail` will never be called
from within the wrapped instance. Therefore this method must be explicitly
invoked from `sendEmail` in `VirusDetector`. Not only does it complicate the code
but it also introduces some inconsistency related to the semantics of `validateEmail`.
On one hand it may be desirable to implement the virus scan in this method, since
the client uses this method to check whether the email is valid. On the other hand,
the validations are supposed to be invoked from the "deepest" `sendEmail` method in a `UserMail`
implementation. It is the implementations's decision whether to invoke the validations
or not (even if it makes sense to do it always). In the case that the implementation
decides to skip the validations, the `VirusDetector` wrapper will ignore this
decision and will validate the email anyway, possibly throwing an exception if
it detects some virus.

This complication is just one of the unfortunate consequences of object schizophrenia.

In the following paragraph I am explaining that if the platform were equipped
with the concept of traits, the processors could be linked by means of the stacking of traits [LINK]
and the above-mentioned problem could be avoided elegantly.

Besides the general email service, there is also an extension offering sending
faxes by email. This service add-on is available to premium customers only.

<div>
<img src="http://zslajchrt.github.io/resources/faxService.png" width="280" />
</div>

The only method in `UserFaxByMail` sends faxes and has the same signature as
`sendEmail`. The default implementation also works in the context of the current
user account and holds a reference to `MailAccount`. It also implements `UserMail`
by delegation in order to facilitate the composition of `UserFaxByMail` and `UserMail`
interfaces.

On top of the model there is the `AlternatingUserMail` class, whose purpose is
to switch transparently between the two accounts if the user has both. `AlternatingUserMail`
contains two references to two instances of the mail services and the class is implemented
by means of the state pattern.[LINK]

<div>
<img src="http://zslajchrt.github.io/resources/alternating.png" width="280" />
</div>

This class also implements the `UserFaxByMail`
interface, which is only a conditional implementation, since unless the current
mail service implements `UserFaxByMail` the `faxEmail` method throws an exception.
Whether the `faxEmail` is possible to invoke may be determined by `canFaxEmail`.
The positive test, however, does not guarantee that the invocation
of `canFaxEmail` will be pass, since the current mail service in the `AlternatingUserMail`
instance might change in the meantime.

Such design is error-prone, because it requires that the topmost wrapper in a
fax-by-mail service instance be one composing `UserMail` and `UserFaxByMail` interfaces, such as
`DefaultUserFaxByMail`, in order to indicate the presence of both interfaces.
The reason is that `AlternatingUserMail` determines whether a mail service instance
is `UserFaxByMail` by checking its type. It follows that it may happen that the topmost
wrapper of such a mail service is wrapped by another wrapper possibly hiding
the `UserFaxByMail` type.  

<div>
<img src="http://zslajchrt.github.io/resources/mailServiceNoTraitsAll.png" width="600" />
</div>

#####Assembling the Service

The following code sketches how the email service may be assebled from individual
components.

```java
Employee employee = new Employee();
RegisteredUser registeredUser = new RegisteredUser();

// We need to clone the state of both employee and registeredUser
EmployeeAdapter employeeAdapter = new EmployeeAdapter(employee);
UserMail userMail1 = new EmployeeUserMail(employeeAdapter);

RegisteredUserAdapter registeredUserAdapter = new RegisteredUserAdapter(registeredUser);
UserMail userMail2 = new RegisteredUserMail(registeredUserAdapter);

userMail1 = new VirusDetector(userMail1);
userMail2 = new VirusDetector(userMail2);

if (registeredUser.isPremium()) {
    userMail2 = new DefaultFaxByMail(registeredUserAdapter, userMail2);
}

// The type of account is still discoverable from the type of both userMail1 and userMail2

AlternatingUserMail userMail = new AlternatingUserMail(userMail1, userMail2);

// The client must be fixed to AlternatingUserMail through which it can determine whether the service supports fax.

Message msg = new Message();
msg.setRecipients(Arrays.asList("pepa@gmail.com"));
msg.setSubject("Hello");
msg.setBody("Hi, Pepa!");

userMail.sendEmail(msg);

userMail.setCurrent(false);

userMail.sendEmail(msg);
```

It is assumed here that the user owns both types of accounts. The variables
`employee` and `registeredUser` hold the two accounts.

The two accounts are adapted to the the common ground by corresponding adapters
and used as the argument to `EmployeeUserMail` and `RegisteredUserAdapter` constructors.

Both particular service instances userMail1 and userMail2 are wrapped by `VirusDetector`.

If the user is a premium customer, then its email service is wrapped by `DefaultFaxByMail`
to provide the fax-by-mail extension.

Then the two mail service instances inserted into `AlternatingUserMail`, which
is able to switch the accounts in the background. This must be the topmost
wrapper so as to preserve the `FaxByMail` type of the resulting service instance.

The rest of the code creates an email message and switches the accounts.

The complete source code may be viewed [here](https://github.com/zslajchrt/morpheus-tutor/tree/master/src/main/java/org/cloudio/morpheus/mail).

#####Summary
The instance lost the track of the account, which is actually used; the account type is no longer detectable from
the instance's type.

The preservation of the type would loosen the coupling between the instance and its client. For example,
the user interface would be able to adapt itself to the type of the account just on the basis of the mail
service instance's type. Now, the UI would have to consult the state of the instance and retrieve the user account instance
from some getter.

Unfortunately, the UserMail interface has no method for getting the user account. Thus it would
be practically impossible to determine the actual user account type without using some reflection tricks
or introducing a new getUserAccount method in the UserMail interface. Both solutions are bad.

The former would be a plain hack, while the latter purblindly introduces a backward incompatible change to the
general contract declared by the UserMail interface. It would force all UserMail implementations to add the new method
even if it would make no sense for them, since some implementation may not wrap any user account object at all.

Such classes would have to return null as an indication there is no underlying user account object. Moreover,
the return type of the getUserAccount method would have to be Object, since there is no common ground shared
by all potential user accounts; the diversity of user accounts is actually the main presupposition of this
case study.

####Modelling the Service With Traits

####Using Object Metamorphism to model the Service

####Conclusion
