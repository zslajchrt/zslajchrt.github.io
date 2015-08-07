---
layout: post
title: Developing Protean Application With Morpheus - Part 5
comments: true
permalink: developing-protean-applications-part5
---

###Unifying Dissimilar Data

```scala
@dimension
trait PersonPublicCommon {
  def nick: String

  def firstName: String

  def lastName: String

  def email: Option[String]
}
```

```scala
@fragment
trait RegisteredUserPublicCommon extends PersonPublicCommon {

  this: RegisteredUserEntity =>

  def nick = regUser.`public`.nick

  def firstName = regUser.`public`.firstName

  def lastName = regUser.`public`.firstName

  def email = regUser.`public`.email

}
```

```scala
@fragment
trait EmployeePublicCommon extends PersonPublicCommon {

  this: EmployeeEntity =>

  def nick = emp.employeeCode

  def firstName = emp.personalData.firstName

  def lastName = emp.personalData.lastName

  def email = Some(s"$nick@thebigcompany.com")

}
```

```scala
val regUserOrEmpModel =
   parse[(RegisteredUserEntity with RegisteredUserPublicCommon) or
         (EmployeeEntity with EmployeePublicCommon)](true)
```

```scala
val publicPerson = asMorphOf[PersonPublicCommon](regUserOrEmpKernel.~)
println(publicPerson.personData)
println(s"${publicPerson.nick}, ${publicPerson.firstName}, ${publicPerson.lastName}, ${publicPerson.email}")
```
