---
layout: post
title: Developing Protean Application With Morpheus - Part 3
comments: true
permalink: developing-protean-applications-part3
---

```json
{
  "public": {
    "nick": "joe4",
    "firstName": "Joe4",
    "lastName": "Doe4",
    "email": "joe4@gmail.com"
  },
  "license": {
    "premium": true,
    "validFrom": "2015-02-21T00:00:00Z",
    "validTo": "2016-02-21T00:00:00Z"
  }
}
```

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

```scala
case class RegisteredUserPublic(nick: String, firstName: String, lastName: String, email: Option[String])

case class RegisteredUserLicense(isPremium: Boolean, validFrom: Date, validTo: Date)

case class RegisteredUser(publicData: RegisteredUserPublic, license: Option[RegisteredUserLicense])
```

```scala
case class EmployeePersonalData(firstName: String, middleName: Option[String], lastName: String, title: String)

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
@dimension
trait UserProtodata {

  def registeredUserJson: JValue

  def employeeJson: JValue

  def initSources(userId: String)
}
```

```scala
import org.morpheus.FragmentValidator._

@fragment
trait RegisteredUserLoader {
  this: UserProtodata with RegisteredUserEntity =>

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
val regUserKernel = singleton[RegisteredUserEntity]
println(regUserKernel.~.registeredUser) // prints null
```

```scala
val regUserLoaderRef: &[$[RegisteredUserLoader with UserProtodataMock]] = regUserKernel
val regUserLoaderKernel = *(regUserLoaderRef, single[RegisteredUserLoader], single[UserProtodataMock])
```

TODO: include the link to the source code of UserProtodataMock

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
val empLoaderRef: &[$[EmployeeLoader with UserProtodataMock]] = empKernel
// analogous to the previous code
```

```scala
import org.morpheus.FragmentValidator._

@dimension
trait FragmentLoader[F] {

  def load: ValidationResult[F]
}

@fragment
trait RegisteredUserLoader extends FragmentLoader[RegisteredUserEntity]

@fragment
trait EmployeeLoader extends FragmentLoader[EmployeeEntity]
```

```scala
val regUserOrEmpKernel = singleton[RegisteredUserEntity or EmployeeEntity]
val regUserOrEmpLoaderRef: &[$[(RegisteredUserLoader or EmployeeLoader) with UserProtodataMock]] = regUserOrEmpKernel
val regUserOrEmpLoaderKernel = *(regUserOrEmpLoaderRef, single[RegisteredUserLoader], single[EmployeeLoader],
  single[UserProtodataMock])
```

```scala
regUserOrEmpLoaderKernel.~.initSources("5") // 4 = a registered user, 5 = an employee
```

```scala
for (loader <- regUserOrEmpLoaderKernel) {
  println(loader.myAlternative)
  loader.load
}
```

```scala
List(org.cloudio.morpheus.dci.socnet.objects.RegisteredUserLoader, org.cloudio.morpheus.dci.socnet.objects.UserProtodataMock, org.cloudio.morpheus.dci.socnet.objects.RegisteredUserEntity)
List(org.cloudio.morpheus.dci.socnet.objects.EmployeeLoader, org.cloudio.morpheus.dci.socnet.objects.UserProtodataMock, org.cloudio.morpheus.dci.socnet.objects.EmployeeEntity)
```

```scala
val regUserEnt = asMorphOf[RegisteredUserEntity](regUserOrEmpKernel)
println(regUserEnt.registeredUser)
val empEnt = asMorphOf[EmployeeEntity](regUserOrEmpKernel)
println(empEnt.employee)
```

```scala
val regUserOrEmpModel = parse[RegisteredUserEntity or EmployeeEntity](true)
var failedFragments: Option[Set[Int]] = None
val morphStrategy = MaskExplicitStrategy(rootStrategy(regUserOrEmpModel), true, () => failedFragments)
val regUserOrEmpKernel = singleton(regUserOrEmpModel, morphStrategy)
```

```scala
failedFragments = Some((for (loader <- regUserOrEmpLoaderKernel;
                             loaderResult = loader.load
                             if !loaderResult.succeeded;
                             frag <- loaderResult.fragment) yield frag.index).toSet)
```

```scala
val regUserEnt = asMorphOf[RegisteredUserEntity](regUserOrEmpKernel)
val empEnt = asMorphOf[EmployeeEntity](regUserOrEmpKernel)
```

```
Exception in thread "main" org.morpheus.AlternativeNotAvailableException
	at org.morpheus.BridgeAlternativeComposer.convertToHolders(MorphingStrategy.scala:446)
	at org.morpheus.Morpher.makeFragHolders$1(Morpher.scala:27)
	at org.morpheus.Morpher.morph(Morpher.scala:43)
	at org.morpheus.Morpher$.morph(Morpher.scala:116)
	at org.morpheus.Morpher$.morph(Morpher.scala:112)
	at org.morpheus.MorphKernel.make(MorphKernel.scala:91)
	at org.cloudio.morpheus.dci.socnet.objects.PersonSample$.main(Loaders.scala:329)
	at org.cloudio.morpheus.dci.socnet.objects.PersonSample.main(Loaders.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:140)
```

```scala
regUserOrEmpKernel.~.remorph
select[RegisteredUserEntity](regUserOrEmpKernel.~) match {
  case Some(regUserEnt) =>
    println(regUserEnt.registeredUser)
  case None =>
    select[EmployeeEntity](regUserOrEmpKernel.~) match {
      case Some(empEnt) => println(empEnt.employee)
      case None => require(false)
    }
}
```

```
Exception in thread "main" org.morpheus.NoAlternativeChosenException
	at org.morpheus.Morpher.makeFragHolders$1(Morpher.scala:25)
	at org.morpheus.Morpher.morph(Morpher.scala:43)
	at org.morpheus.Morpher$.morph(Morpher.scala:116)
	at org.morpheus.MorphKernel$$anon$2.morph(MorphKernel.scala:110)
	at org.morpheus.MutableMorphContext.proxy$lzycompute(MorphContext.scala:297)
	at org.morpheus.MutableMorphContext.proxy(MorphContext.scala:288)
	at org.morpheus.MorphKernel.mutableProxy_(MorphKernel.scala:114)
	at org.morpheus.MorphKernel.make_$tilde(MorphKernel.scala:96)
	at org.morpheus.MorphKernel.$tilde$lzycompute(MorphKernel.scala:84)
	at org.morpheus.MorphKernel.$tilde(MorphKernel.scala:84)
	at org.cloudio.morpheus.dci.socnet.objects.PersonSample$.main(Loaders.scala:333)
	at org.cloudio.morpheus.dci.socnet.objects.PersonSample.main(Loaders.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:140)
```

```scala
try {
  regUserOrEmpKernel.~.remorph
  select[RegisteredUserEntity](regUserOrEmpKernel.~) match {
    case Some(regUserEnt) =>
      println(regUserEnt.registeredUser)
    case None =>
      select[EmployeeEntity](regUserOrEmpKernel.~) match {
        case Some(empEnt) => println(empEnt.employee)
        case None => require(false)
      }
  }
}
catch {
  case ae: NoViableAlternativeException =>
    println("Cannot load registered user or employee")
}
```


####Case Study Overview

 - A popular web site like a search engine
 - In the beginning it has no clue about its users
 - Registration of users to improve search results and targeting ads
 - At this point the user entity consists of a few simple attributes like ID, email, name...
   However the db is populated with hundreds of millions of entities
 - The company starts asking users for private details like the phone number or address
   under the pretext that they needed to provide users with premium services. The entity contains the private info section.
 - The company starts tracking the activity of users, the entity is extended by hobbies for instance
 - The company unleashes its bots to retrieve additional information about its users from other sites like LinkedIn.
   The entity now contains professional career data.
 - The company offers registered users with a free email service. The entity contains contacts.
 - Since the company keeps track who is online and knows the contacts of many of its users it can
   implement a chat service.


####Data Description

####Modeling Protean Data

####Instantiating and Loading

####Protean Behavior (Offline/Online)

####Protean Networks

####Networks Mapper

####Colleagues Network

####Friends Network

####Application of Friends Network

####Simultaneous Networks
