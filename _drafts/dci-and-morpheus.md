---
layout: post
title: DCI And Morpheus
comments: true
permalink: dci-and-morpheus
---

###What is DCI?

**Data, context, interactions** (DCI) is a software paradigm whose goal is to bring the end user's mental models and computer program models closer together. To put it in a nutshell, the user must feel that he or she directly manipulates the objects in computer memory that correspond to the images in his or her head.

Data, context and interactions are the three fundamental facets of the end user's interpretation of computer data. Data itself are nothing more than bits in program memory. Only after the data are put into a context, in which they are subjects to interactions between them, the data can yield some information.

The paradigm was invented by Trygve Reenskaug, who is also the inventor of the MVC pattern. DCI can be seen as its further development.


###Background

DCI is anchored in object-oriented programming (OOP), however it must cope with some inherent flaws of OOP. It is generally accepted that OOP is very good at capturing the system's state by means of classes and their properties. OOP is also good at expressing operations with the state captured by a class, unless these operations involve some kind of collaboration with instances of other classes.

However, OOP fails to express collaborations between objects. These collaborations we call use cases. An object may appear in several use cases and it may behave quite differently in each use case. Because of the lack of another concept in OOP we are forced to express such a use-case-specific behavior as an operation in a class. And it has several bad consequences:
   1. There is no single file or other artifact dedicated solely to one use case, where we could see all interactions between objects. It makes the orientation in the code and its maintenance pretty difficult.
   2. The whole behavior of the use case is scattered across the classes of the collaborating objects. It leads us to add a number of unrelated methods to classes with every new use case (causing higher coupling and lower cohesion of classes).
   3. It is practically impossible to separate the stable part of the code, i.e. that which captures the data, from the variable part, i.e. the use cases (behavioral part).

See [this article](http://www.sitepoint.com/dci-the-evolution-of-the-object-oriented-paradigm/) by Victor Savkin, which explain nicely the problem of collaboration in OOP.


When transferring money the user's head probably *contains* two things: the two accounts selected to take part in the transaction. Besides the two things/objects he or she also knows the roles of each account: one as the source and the other as the destination account.

![Bank Accounts](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-2.png "Bank Accounts")

![Transfer Money Use-Case](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-1.png "Transfer Money Use-Case")


![Money Transfer](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-3.png "Money Transfer")

```scala
val x = 1
```
