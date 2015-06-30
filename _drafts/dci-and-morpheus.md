---
layout: post
title: DCI And Morpheus
comments: true
permalink: dci-and-morpheus
---

###What is DCI?

DCI is a software paradigm whose goal is to bring the end user's mental models and computer program models closer together. To To put it in a nutshell, the user must feel that he or she directly manipulates the objects in computer memory that correspond to the images in his or her head.

The paradigm was invented by Trygve Reenskaug, who is also the inventor of the MVC pattern. DCI can be seen as its further development.

The acronym means Data-Context-Interactions and it expresses the three fundamental components of the end user's interpretation of computer data. Data itself are nothing more than bits in program memory. Only after the data are put into a context, in which they are subjects to interactions between them, the data can yield some information.

###Background

Although it is anchored in object-oriented programming it must cope with some inherent flaws of OOP.




When transferring money the user's head probably *contains* two things: the two accounts selected to take part in the transaction. Besides the two things/objects he or she also knows the roles of each account: one as the source and the other as the destination account.

![Bank Accounts](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-2.png "Bank Accounts")

![Transfer Money Use-Case](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-1.png "Transfer Money Use-Case")


![Money Transfer](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-3.png "Money Transfer")

```scala
val x = 1
```
