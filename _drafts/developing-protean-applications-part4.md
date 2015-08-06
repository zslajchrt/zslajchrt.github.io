---
layout: post
title: Developing Protean Application With Morpheus - Part 4
comments: true
permalink: developing-protean-applications-part4
---

```scala
@fragment
trait PersonPublicCommon {

  this: RegisteredUserEntity or EmployeeEntity =>

  def nick: String = ???

  def firstName: String = ???

  def lastName: String = ???

  def email: Option[String] = ???

}
```

```scala
sealed trait PersonPublic
case class RegisteredUserPublic(nick: String, firstName: String, lastName: String, email: Option[String]) extends PersonPublic
case class Employee(employeeCode: String, position: String, department: String, personalData: EmployeePersonalData) extends PersonPublic
```

```scala
@fragment
trait PersonPublicCommon {

  this: RegisteredUserEntity or EmployeeEntity =>

  lazy val personData: PersonPublic = {
    List(
      for (pp <- select[RegisteredUserEntity](this)) yield pp.registeredUser.`public`,
      for (pp <- select[EmployeeEntity](this)) yield pp.employee
    ).find(_.isDefined).get.get
  }

...

```

```scala
def nick = personData match {
  case RegisteredUserPublic(n, _, _, _) => n
  case Employee(code, _, _, _) => code
}

def firstName = personData match {
  case RegisteredUserPublic(_, fn, _, _) => fn
  case Employee(_, _, _, EmployeePersonalData(fn, _, _, _)) => fn
}

def lastName = personData match {
  case RegisteredUserPublic(_, _, ln, _) => ln
  case Employee(_, _, _, EmployeePersonalData(_, _, ln, _)) => ln
}

def email: Option[String] = personData match {
  case RegisteredUserPublic(_, _, _, em) => em
  case Employee(code, _, _, _) => Some(s"$code@thebigcompany.com")
}
```

TODO: Include a link to the source of PersonPublicCommon

```scala
val regUserOrEmpModel = parse[PersonPublicCommon with (RegisteredUserEntity or EmployeeEntity)](true)
```

```scala
val publicPerson = asMorphOf[PersonPublicCommon](regUserOrEmpKernel.~)
println(publicPerson.personData)
println(s"${publicPerson.nick}, ${publicPerson.firstName}, ${publicPerson.lastName}, ${publicPerson.email}")
```


```scala
@dimension
trait MailService {
  def send(message: Message): Unit
}

case class Attachment(name: String, data: Array[Byte], mime: String)

case class Message(from: String, recipients: List[String], subject: String, message: String, attachments: List[Attachment])
```
