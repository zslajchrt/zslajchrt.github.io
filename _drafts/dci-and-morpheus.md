---
layout: post
title: DCI And Morpheus
comments: true
permalink: dci-and-morpheus
---

###What is DCI?

**Data, context, interactions** (DCI) is a software paradigm whose goal is to bring the end user's mental models and computer program models closer together. To put it in a nutshell, the user must feel that he or she directly manipulates the objects in computer memory that correspond to the images in his or her head.

Data, context and interactions are the three fundamental facets of the end user's interpretation of computer data. Data itself are nothing more than bits in program memory. Only after the data are put into a context, in which they are subjects to interactions between them, the data can yield some information.

The paradigm was invented by Trygve Reenskaug, who is also the inventor of the MVC pattern. DCI can be seen as its further development. See [this article](http://www.artima.com/articles/dci_vision.html) describing the DCI vision.


###Background

DCI is anchored in object-oriented programming (OOP), however it must cope with some inherent flaws of OOP. It is generally accepted that OOP is very good at capturing the system's state by means of classes and their properties. OOP is also good at expressing operations with the state captured by a class, unless these operations involve some kind of collaboration with instances of other classes.

However, OOP fails to express collaborations between objects. These collaborations we call use cases. An object may appear in several use cases and it may behave quite differently in each use case. Because of the lack of another concept in OOP we are forced to express such a use-case-specific behavior as an operation in a class. And it has several bad consequences:
   1. There is no single file or other artifact dedicated solely to one use case, where we could see all interactions between objects. It makes the orientation in the code and its maintenance pretty difficult.
   2. The whole behavior of the use case is scattered across the classes of the collaborating objects. It leads us to add a number of unrelated methods to classes with every new use case (causing higher coupling and lower cohesion of classes).
   3. It is practically impossible to separate the stable part of the code, i.e. that which captures the data, from the variable part, i.e. the use cases (behavioral part).

See [this article](http://www.sitepoint.com/dci-the-evolution-of-the-object-oriented-paradigm/) by Victor Savkin, which explain nicely the problem of collaboration in OOP.

###Solution

DCI represents every use case by means of the so-called *context*. The context defines *roles* performing *interactions* between themselves. Each role in the context is played by one corresponding object (data, entity). The role contains the code that would otherwise reside in the object's class. Thus, roles effectivelly separate the stable part of the code from the unstable.

The context itsels only defines roles and triggers the use-case. The Context and roles should reside in one dedicated file so that one could easily investigate the interactions.

###Example

The paradigmatical example of DCI is a simulation of a **money transfer**. It is simple enough to illustrate the fundamentals of DCI.

The use case scenario is this: the end user uses the bank terminal to transfer money from one account to another. He or she selects the source and the destination accounts from the list of accounts. Then he or she specifies the amount of money to be transfered and starts the transaction. Some exceptions can be raised, of course, for instance as long as there is not enough balance in the source account to perform the transfer.

For the sake of simplicity, let us assume that the data model of the bank application is just the list of the end user's accounts. Every account is represented by an object encapsulating some basic properties like the `balance` along with some basic operations like `increaseBalance` and `decreaseBalance`. We can expect that such a data model will be pretty alligned with the end user's mental model.

![Bank Accounts](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-4.png "Bank Accounts")

When transferring money the user will intuitively be familiar with the basic steps of the procedure. He or she will know that it is a simple interaction between two accounts, one playing the role of the source account and the other playing the destination account role and that the balance of the source account will be decreased by the amount of the transfered money while the destination's balance will be increased by the same amount.

In DCI it is quite easy to express both the data model and the interactions accordingly to the user's mental model and intuition.

two things: the two accounts selected to take part in the transaction. Besides the two things/objects he or she also knows the roles of each account: one as the source and the other as the destination account.

![Transfer Money Use-Case](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-1.png "Transfer Money Use-Case")

![Chosen Bank Accounts](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-2.png "Chosen Bank Accounts")

![Money Transfer](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-3.png "Money Transfer")

```scala
val x = 1
```
