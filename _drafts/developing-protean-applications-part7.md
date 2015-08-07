---
layout: post
title: Developing Protean Application With Morpheus - Part 7
comments: true
permalink: developing-protean-applications-part7
---

###Protean Behavior Caused By Processed Data

```scala
@dimension
@wrapper
trait AttachmentValidator extends MailService {

  abstract override def send(message: Message): Unit = {
    checkAttachments(message.attachments)

    super.send(message)
  }

  def checkAttachments(attachments: List[Attachment]): Unit = {
    // todo
    println(s"Checked ${attachments.size} attachments")
  }

}
```

```scala
val mailRef: &[$[MailService with
  FromHeaderValidator with
  /?[SignatureAppender] with
  \?[AttachmentValidator]]] = regUserOrEmpKernel
```

TODO: Make a point on the difference between \? and /? and its influence
on the resulting morph

```scala
val mailKernel = *(mailRef,
  single[MailServiceMock],
  single[FromHeaderValidator],
  single[SignatureAppender],
  single[AttachmentValidator])
```

```scala
object MailServiceStrategy {

  def apply(msg: Message): MorphingStrategy[mailKernel.Model] = {
    val hasAtt: Option[Int] = if (msg.attachments.nonEmpty) Some(0) else None
    promote[AttachmentValidator](rootStrategy(mailKernel.model), hasAtt)
  }
}
```

```scala
def sendMail(msg: Message): Unit = {
  val m = mailKernel.~.remorph(MailServiceStrategy(msg))
  println(m.myAlternative)
  m.send(msg)
}
```

```scala
// no attachment
sendMail(Message(None, List("agata@gmail.com"), "Hello", "Bye", Nil))
// some attachment
sendMail(Message(None, List("agata@gmail.com"), "Hello", "Bye", List(Attachment("att1", Array[Byte](0,1,2), "mime1"))))
```
