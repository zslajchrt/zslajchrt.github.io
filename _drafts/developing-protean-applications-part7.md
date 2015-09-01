---
layout: post
title: Developing Protean Application With Morpheus - Part 7
comments: true
permalink: developing-protean-applications-part7
---

###Modelling Protean Service With Dynamic Traits

```groovy
@CompileStatic
trait Employee {

    private String firstName;
...
}
```

```groovy
def employee = new Object() as Employee;
loadEmployee(employeeData);
def regUser = new Object() as RegisteredUser;
loadRegUser(regUserData);
```

```groovy
UserMail employeeMail = employee.withTraits(EmployeeAdapter, DefaultUserMail, VirusDetector)
```

```groovy
UserMail regUserMail = regUser.withTraits(RegisteredUserAdapter, DefaultUserMail, VirusDetector);
UserMail regUserMailPremium = regUserMail.withTraits(Premium, DefaultFaxByMail);
```

```groovy
AlternatingUserMail userMail = new AlternatingUserMail() {
    @Override
    UserMail getDelegate() {
        Calendar c = Calendar.getInstance();
        def h = c.get(Calendar.HOUR_OF_DAY);
        if (h >= 8 && h < 17) {
            return getEmployeeMail();
        } else {
            return getRegUserMail();
        }
    }

    UserMail getEmployeeMail() {
        return employeeMail;
    }

    UserMail getRegUserMail() {
        if (regUser.premium &&
                regUser.validTo != null &&
                regUser.validTo.toCalendar().after(Calendar.getInstance()))
            return regUserMailPremium;
        else
            return regUserMail;
    }

};
```

- combinatorial explosion is over, the

- weak type system, cannot use composite types for variables:
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

###Generalization

modeling multidimensional data, combinatorial explosion of class declarations,
multidimensional polymorphism -> a need to describe all combinations in one expression

a loss-less extension of types, the information about the types constituting
an object's class should percolate through all abstraction layers,
from the persistence layer, through the business layer to the presentation layer,
loose coupling,

cloning object state

the client is tightly coupled with a concrete implementation of a general interface:
to find out the type of the underlying entity, to determine the implemented
interfaces (isXXX methods)

The functionality of AlternatingUserMail should be provided by the platform.
