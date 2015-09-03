---
layout: post
title: Developing Protean Application With Morpheus - Part 8
comments: true
permalink: developing-protean-applications-part8
---

###Wrapping Up

In the light of the above-mentioned findings,... strong



####Type Preservation
Why?


####Multidimensionality

TODO: What is it?
An instance may assume any of the set of predefined forms. Each form corresponds
to a point in a multidimensional space, where each dimension is represented by
an abstract "trait" (material) and values in the dimension are "concrete" traits (paper, metal ...).

modeling multidimensional data, combinatorial explosion of class declarations,
multidimensional polymorphism -> a need to describe all combinations in one expression

####Metamorphism

TODO: What is it? It

####Multidimensional Metamorphism

TODO: What is it?

a loss-less extension of types, the information about the types constituting
an object's class should percolate through all abstraction layers,
from the persistence layer, through the business layer to the presentation layer,
loose coupling,

cloning object state

the client is tightly coupled with a concrete implementation of a general interface:
to find out the type of the underlying entity, to determine the implemented
interfaces (isXXX methods)

The functionality of AlternatingUserMail should be provided by the platform.

####A Vision of New Platform

In order to find a solution we must not only take into account
the generation, but also consider some extension of the type system.

The solution stems from the idea to express the multidimensionality of a domain
object as a type. It is actually an extension of the traditional way of expressing
object types. For example the following expression refers to a simple type:

```scala
  Thing
```

and this expression refers to a composition of three type.

```scala
  Thing with Paper with Cylinder
```

It would be nice if another type expression could be constructed to refer to a type
containing alternatives (a sort of union).

```scala
  Thing with (Paper or Metal) with (Rectangle or Cylinder)
```

The previous type expresses in one line all combinations of forms that an item
can assume. Adding a new dimension or a new type to an existing dimension does
not cause a code explosion.

To support such a kind of expression in applications, there must be some extension
in the language platform, especially in the compiler and type system sections.

The following code shows how such a complex type could be used to instantiate
an item (the code uses the real *Morpheus* code):

```scala
  val itemKernel = compose[Thing with (Paper or Metal) with (Rectangle or Cylinder)]
  val item = itemKernel.morph(new ItemBuilder(scanEvent)) // ItemBuilder determines the final form of the item

  assert(true, item.isInstanceOf[Thing with Material with Shape])

  val maybeBanknote = item.isInstanceOf[Paper with Rectangle] // true or false
```

Without delving too much into the details, the program flow can be described
this way:

First, the `compose` operator creates the so-called kernel for the type specified
in the brackets. The kernel 'knows' all the alternative forms which an item
can assume. It also holds factories for instantiating individual fragments, of
which the type is composed.

Second, the item is instantiated by invoking `morph` method on the kernel.
Since there are a number of possible alternatives, the kernel needs a companion,
which helps him to determine the right alternative. The `ItemBuilder` is such
a companion which selects the correct alternative according to the `scanEvent`
passed as the argument to its constructor.

Third, the returned object is always an instance of type `Thing with Material with Shape`,
which is the lowest common ancestor type of all alternative types.

Fourth, the item can be checked whether it is a certain combination of
concrete types by means of `isInstanceOf` method as usual.

The complete code including the initialization is discussed later.

For the sake of further analysis let us assume that at this stage the multidimensional
protodata can be modeled by means of the extended system system sketched above.


#####Alternative Instantiation Process

The traditional approach in all class-oriented languages follows this instantiation
schema:

```
   class -> class instance
```

In other words, the class precedes its instances, which sounds logical.

However, it is possible to consider other, a more general schema, which introduces
the concept of *fragment*. The fragment can be seen as a weaker version of
the class. It may or may not be concrete, however, even if it is abstract, it can
be instantiated in contrast to an abstract class. Such abstract fragment instances
can have state but cannot be used independently.

The alternative instantiation schema employs fragments and is expressed in the following
causal chain:

```
   fragments -> fragment instances -> composite class -> composite class instance
```

The schema can interpreted as follows: Fragments and possibly also their
instances exist in the system even before there exists any class. All classes
in the system are compositions of one or more fragments and all instances of
such classes are compositions of corresponding fragment instances.

Let us examine the behavior of the proposed mechanism in the simplest case:

```
   x = instantiate X("Hello")
```

The `instantiate` procedure starts with looking for fragment `X` in the system. The system
responds by returning a factory for obtaining an instance of `X`, otherwise an exception
is raised. Then the factory returns the instance, which is immediately returned as
the result of the procedure. The procedure assumes that the compiler has already
done all necessary checks so that it can simply return the fragment instance.
In this case there is no need for generating a composite class since there is
only one fragment involved in the instantiation.

Let us examine now a more complex example.

```
  xy = instantiate X("Hello"), Y
```

Now the procedure starts looking for fragments `X` and `Y` and obtains their instances.
Since now the target class is a composition of `X` and `Y`, the class loader is asked
to load and possibly generate the class. Then an instance of this class is composed
from the instances of fragments `X` and `Y`. Since there is no constructor argument
list specified for Y it can be interpreted as a signal to not create a new instance
and instead to use a cached fragment instance (e.g. a singleton).

Note: Fragments should be considered having no identity until after the instantiation
procedure finishes.

#####Object Identity Deprivation

Once the existence of all components is confirmed, the new object can be made
up of its components. The constituting components are deprived of their identity
in behalf of the newly composed entity. It is necessary in order to prevent
the object schizophrenia.

Unfortunately, the described process of object identity deprivation is impossible
or very difficult to carry out, especially in statically typed languages ([here]((http://www.artima.com/articles/dci_vision.html)) or [here](http://www.sitepoint.com/dci-the-evolution-of-the-object-oriented-paradigm/)).
Once an object is instantiated it keeps its identity until its destruction. There
is no concept of 'identity deprivation' or 'getting rid of identity'.
The only way to compose an object of other already existing objects is to use
delegation, which immediately leads to object schizophrenia and thus to the inability
to determine the object's character from its type.

Therefore, to make the direct mapping based on type modeling feasible it is
necessary to rethink the process of instantiation in programming languages and
and come up with some alternative.
