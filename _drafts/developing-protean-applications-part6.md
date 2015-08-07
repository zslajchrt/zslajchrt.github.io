---
layout: post
title: Developing Protean Application With Morpheus - Part 6
comments: true
permalink: developing-protean-applications-part6
---

###Protean Behavior Caused By Underlying Data

```scala
@dimension
trait MailService {
  def send(message: Message): Unit
}

case class Attachment(name: String, data: Array[Byte], mime: String)

case class Message(from: String, recipients: List[String], subject: String, message: String, attachments: List[Attachment])
```

```scala
@dimension
@wrapper
trait FromHeaderValidator extends MailService {
  this: PersonPublicCommon =>

  abstract override def send(msg: Message): Unit = {
    val msgWithFrom = msg.from match {
      case None => msg.copy(from = this.email)
      case Some(f) => msg
    }
    super.send(msgWithFrom)
  }

}
```

```scala
val mailRef: &[$[MailService with FromHeaderValidator]] = regUserOrEmpKernel
val mailKernel = *(mailRef, single[MailServiceMock])
```

TODO: Link to MailServiceMock

```scala
val msg = Message(None, List("agata@gmail.com"), "Hello", "Bye", Nil)
mailKernel.~.send(msg)
```

```scala
@dimension
@wrapper
trait SignatureAppender extends MailService {
  this: EmployeeEntity =>

  abstract override def send(msg: Message): Unit = {
    super.send(msg.copy(message = msg.message + signature))
  }

  lazy val signature: String = {
    s"""
     ${emp.personalData.title} ${emp.personalData.firstName} ${emp.personalData.lastName}
     ${emp.position}
     ${emp.department}
     TheBigCompany.com
     http://thebigcompany.com
     """
  }

}
```

```scala
val mailRef: &[$[MailService with FromHeaderValidator with /?[SignatureAppender]]] = regUserOrEmpKernel
val mailKernel = *(mailRef, single[MailServiceMock], single[FromHeaderValidator], single[SignatureAppender])
```

TODO: describe the effect of \?[F] and /?[F] on choosing the winning alternative
