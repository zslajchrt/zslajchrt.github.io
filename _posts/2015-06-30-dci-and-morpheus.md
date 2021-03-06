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

See [this article](http://www.sitepoint.com/dci-the-evolution-of-the-object-oriented-paradigm/) by Victor Savkin, which explains nicely the problem of collaboration in OOP.

###Solution

DCI represents every use case by means of the so-called *context*. The context defines *roles* performing *interactions* between themselves. Each role in the context is played by one corresponding object (data, entity). The role contains the code that would otherwise reside in the object's class. Thus, roles effectivelly separate the stable part of the code from the unstable.

The context itsels only defines roles and triggers the use-case. The Context and roles should reside in one dedicated file so that one could easily investigate the interactions.

###Example

The paradigmatical example of DCI is a simulation of a **money transfer**. It is simple enough to illustrate the fundamentals of DCI.

The use case scenario is this: the end user uses the bank terminal to transfer money from one account to another. He or she selects the source and the destination accounts from the list of accounts. Then he or she specifies the amount of money to be transfered and starts the transaction. Some exceptions can be raised, of course, for instance as long as there is not enough balance in the source account to perform the transfer.

For the sake of simplicity, let us assume that the data model of the bank application is just the list of the end user's accounts. Every account is represented by an object encapsulating some basic properties like the `balance` along with some basic operations like `increaseBalance` and `decreaseBalance`. We can expect that such a data model will be pretty alligned with the end user's mental model.

![Bank Accounts](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-4.png "Bank Accounts")

When transferring money the user will intuitively be familiar with the basic steps of the procedure. He or she will know that it is a simple interaction between two accounts, one playing the role of the source account and the other playing the destination account role and that the balance of the source account will be decreased by the amount of the transfered money while the destination's balance will be increased by the same amount.

![Transfer Money Use-Case](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-1.png "Transfer Money Use-Case")

It is important to note that the roles, i.e. the triangle and the rectangle have no identity. They can be regarded as costumes. It is the object playing the role that carries the identity.

In many object oriented programming languages it is usually pretty easy to express both the data and the abstract context (i.e. using roles that are not bound to the objects yet). What is not that easy, however, is the binding of objects to their respective roles in the context. In this use case the goal is to fill the holes in the rectangle and the triangle by the chosen account objects.

![Chosen Bank Accounts](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-2.png "Chosen Bank Accounts")

![Money Transfer](https://raw.githubusercontent.com/zslajchrt/morpheus/master/src/main/doc/pict/dci-transfer-money-3.png "Money Transfer")

This is the moment at which the things are becoming complex. Without modern programming concepts like mixins, traits, aspects or meta-programming we would hardly overcome this point. And yet any of these techniques has its own issues and does not perfectly fit to DCI.

See [this article on Wikipedia](https://en.wikipedia.org/wiki/Data,_context_and_interaction) dealing with various issues, or go [here to learn what *self schizophrenia*](https://en.wikipedia.org/wiki/Schizophrenia_(object-oriented_programming)) is.

So let us suppose we have managed to get through this difficult step and the objects are assigned to their roles. Now the context is ready to transfer the money. The user presses the **Go** button emitting a command that is delegated to the context, which will enact the use case.

### Using Morpheus

[Morpheus](https://github.com/zslajchrt/morpheus) is a Scala extension that makes possible for an object to change its *shape* dynamically. Unsurprisingly, it can be used to implement a DCI application.

You can consult or check out the source code of this example [here](https://github.com/zslajchrt/morpheus-tutor/tree/master/src/main/scala/org/cloudio/morpheus/dci).


#### Modelling Data

Let us begin with the data model. Actually, there is nothing specific to Morpheus. We are only required to model the entities as traits, not classes.

```scala
trait Account {
  def Balance: BigDecimal

  def decreaseBalance(amount: BigDecimal): Unit

  def increaseBalance(amount: BigDecimal): Unit
}

class AccountBase(initialBalance: BigDecimal) extends Account {

  private var balance: BigDecimal = initialBalance

  def Balance = balance

  def decreaseBalance(amount: BigDecimal) = balance -= amount

  def increaseBalance(amount: BigDecimal) = balance += amount
}

class SavingsAccount(initialBalance: BigDecimal) extends AccountBase(initialBalance) {

}

class CheckingAccount(initialBalance: BigDecimal) extends AccountBase(initialBalance) {

}

class RetiringAccount(initialBalance: BigDecimal) extends AccountBase(initialBalance) {

}
```

#### Modelling Context

The next DCI facet is the context defining the interacting roles. In this case the context defines only two roles: `Source` and `Destination`. The amount of money to be transferred is also included in the context (as a stage prop).

```scala
trait Context {
  private[moneyTransfer] val Source: Account with Source
  private[moneyTransfer] val Destination: Account with Destination
  val Amount: BigDecimal
}
```

The context is a simple trait declaring the two roles as values of a composite type `Account with Source`, resp. `Account with Destination`. The types indicate that the members **are** accounts playing the corresponding roles in the use case. `Source` and `Destination` are roles defined below.

The implementation of the context is also pretty simple. It uses the `role` macro that hides some Morpheus boilerplate. The macro accepts three types: the role type, the object type and the context type. The only argument is the reference to the object.

```scala
class ContextImpl(srcAcc: Account, dstAcc: Account, val Amount: BigDecimal) extends Context {

  private[moneyTransfer] val Source = role[Source, Account, Context](srcAcc)
  private[moneyTransfer] val Destination = role[Destination, Account, Context](dstAcc)

  def trans(): Unit = {
    Source.transfer
  }

}
```

The following class is the same as the previous one, however, now wihout the `role` macro.

```scala
class ContextImpl(srcAcc: Account, dstAcc: Account, val Amount: BigDecimal) extends Context {

  private[moneyTransfer] val Source = {
    implicit val dataFrag = external[Account](srcAcc)
    implicit val selfFrag = external[Context](this)
    singleton[Account with Source with Context].!
  }

  private[moneyTransfer] val Destination = {
    implicit val dataFrag = external[Account](dstAcc)
    implicit val selfFrag = external[Context](this)
    singleton[Account with Destination with Context].!
  }

  def trans(): Unit = {
    Source.transfer
  }

}
```

Using the `singleton` macro allows us to join the account object, the role and the context into one instance.

Note: The `singleton` macro can do much more things, this is only the most basic functionality.

#### Modelling Interactions (Roles)

The roles are also modelled as traits. Now we have to mark them with the `fragment` annotation. Both the bound account object and the context are available through the self-type `Account with Context`.

```scala
@fragment
trait Source {
  this: Account with Context =>

  private def withdraw(amount: BigDecimal) {
    decreaseBalance(amount)
  }

  def transfer {
    Console.println("Source balance is: " + Balance)
    Console.println("Destination balance is: " + Destination.Balance)

    Destination.deposit(Amount)
    withdraw(Amount)

    Console.println("Source balance is now: " + Balance)
    Console.println("Destination balance is now: " + Destination.Balance)
  }

}

```

```scala
@fragment
trait Destination {
  this: Account with Context =>

  def deposit(amount: BigDecimal) {
    increaseBalance(amount)
  }
}
```

It is important to note that the context and its roles reside in the same file.

#### Running the Program

And finally we can run the program.

```scala
object App {

  def main(args: Array[String]): Unit = {
    val ctx = new ContextImpl(new AccountBase(10), new AccountBase(50), 5)
    ctx.trans()
  }

}
```

### Links
   * [This example](https://github.com/zslajchrt/morpheus-tutor/tree/master/src/main/scala/org/cloudio/morpheus/dci)
   * [Morpheus](https://github.com/zslajchrt/morpheus)
   * [Morpheus Tutorial](https://github.com/zslajchrt/morpheus-tutor)
   * [DCI Vision](http://www.artima.com/articles/dci_vision.html)
   * [DCI in Ruby and nice explanations](http://www.sitepoint.com/dci-the-evolution-of-the-object-oriented-paradigm/)
   * [DCI on Wikipedia](https://en.wikipedia.org/wiki/Data,_context_and_interaction)
   * [The Roots of DCI](http://fulloo.info/Documents/2010DCI-Origin.pdf)
   * [Squeak Examples](http://fulloo.info/Examples/SqueakExamples/)
   * [Object Schizophrenia](http://users.jyu.fi/~sakkinen/inhws/papers/Sekharaiah.pdf), [Self Schizophrenia on Wikipedia](https://en.wikipedia.org/wiki/Data,_context_and_interaction)
   * [DCI Google Group](https://groups.google.com/forum/#!topic/object-composition/2F6kwUpb0FQ)
