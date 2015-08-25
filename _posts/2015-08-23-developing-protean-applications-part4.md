---
layout: post
title: Developing Protean Application With Morpheus - Part 4
comments: true
permalink: developing-protean-applications-part4
---

###New Context Domain For Protodata

Now we turn our attention back to the airport scanner protodata and the effort
to find some use for it.

Let us assume that someone comes up with an idea to build an application performing
some analytics of the luggage contents and publishing the insights via a web portal
to customers, which could be for example the statistics bureau, economic chambers
or customs.

As the first analysis the application will publish the aggregate amount of cash
detected in luggage per some time period.

Such a task requires that the application be able to recognize coins and banknotes
among luggage items. Considering the scanner protodata contains information
about the shape and material only, it seems inevitable to use other source to
detect reliably coins and banknotes. Such a source can be a database of
international currencies containing comprehensive information about current and
historical world currencies including physical properties. (Note: There is a limitation
in this scenario, since all U.S. banknotes have the same dimensions. The database
could not recognize individual dollar banknotes just on the basis of their size).

The domain model of the new context is depicted on the following diagram:

Figure 5: The context domain diagram
<div>
<img src="http://zslajchrt.github.io/resources/currencyModel.png" width="280" />
</div>

The basic logic of the currency flow analysis is very simple. It scans a segment
of the protodata corresponding to the selected time period and it attempts to
convert each luggage item into a currency. Such a set of currency objects is then
aggregated into a statistical report and published on the portal.

The only missing part is now the conversion of the item to a currency object. This
task is analyzed in the next paragraph.

#####Mapping the context domain on the proto-domain

The goal of the mapping is to bind the objects from the target domain to
the objects from the source domain in order to avoid any intermediary processes
normalizing and transforming the source domains objects into the target domain ones.

Furthermore, since the type systems of advanced programming languages are
powerful enough to grasp the complex nature of domain objects, domain models
should use primarily the type system to model the objects.

Here, the source domains are the scanner protodata and the currency database, while
the target domain is the currency context. The mapping declares how an object
from the target domain can be bound to objects from the source domains. Particularly,
the mapping in this example contains rules for mapping coins and banknotes to
luggage items and records in the currency database.

The mapping between the currency and luggage items domain is sketched on the following diagram.

Figure 6: Mapping context domain to proto-domain diagram
<div>
<img src="http://zslajchrt.github.io/resources/itemCurrencyMap.png" width="450" />
</div>

Each concrete type in the target domain is mapped to a subset of source domain
concrete types. Moreover, this mapping is *orthogonal* since all these
subsets do not overlap with one another.

Since the mapping should utilize primarily the capabilities of the underlying
type system, the rules should be declared by means of type expressions
composed of the trait types defined in the source and target domains.

Let us try to formally declare the mapping rules for coins and banknotes:

```
  Coin -> {Metal, Cylinder}
  Banknote -> {Paper, Rectangle}
```

In other words, these rules say that a coin is a metal cylinder and a banknote
is a paper rectangle. Of course, such definitions are far from complete. Not
every metal cylinder is a coin.

This is the moment, when the auxiliary currency database is used to complete the missing
parts in the definitions.

We can declare that a metal cylinder is a coin if the currency database contains
a coin record, whose physical properties correspond to that of the metal cylinder,
of course, within some predefined margin of error.

To reflect these additional constraints the Coin mapping rule can be completed as follows:

```
  Coin -> {m: Metal, c: Cylinder, ce: CoinExemplar(radius = c.diameter/2, thickness = c.height)}
```

The rule for Coin is now a composition of three components and can be seen as
a template for valid Coin instances demanding existence of the three constituting
components.

If an item being mapped to a coin is a metal cylinder, then the existence of
the first two components is fulfilled automatically. However, the existence of
the third component must be confirmed by a lookup in the currency database.

Every component is annotated with an identifier, which can be used in expressions
constraining property values in other components such as `radius` and `thickness` in
CoinExemplar.

*Note: Since the resulting object is a composition of three existing object from the source
domains there is no lost of information, even if the target object exposes only
some of it. The complete information is encapsulated in the target object and
some future minor changes in the domain code can make accessible.*

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

#####Summary

A new context must be discovered in order for "useless" protodata to yield some value

The data objects in the protodata may manifest the so-called *multidimensional polymorphism*.
Multidimensional polymorphism means that the set of all forms, which an object can assume,
is equivalent to a cartesian product of the so-called dimensions. A dimension
represents an abstract trait of the object and consists of types implementing
this trait.

The new context may found by evaluating the protodata against a new data source - the auxiliary data

The domain of the new context and the proto-domain can be structurally very
distant, possibly orthogonal.

The protodata and the auxiliary data must be somehow adapted for the new context domain

Transformation and normalization are lossy processes, thus they remove data, which
the application could use in the future. Next, such a processing may cause delays
and put additional burden on the infrastructure. In the course of time the transforming
processes tend to become unmaintainable.

The solution is to map the new context domain directly to the source domains and
to compose the target domain objects of the objects from the source domains.

The mapping and composition should be the task for the so-called mapper.

The mapper should be domain-agnostic, i.e. its functionality should
not be principally driven by the knowledge of the participating domains.
In other words, the mapper should be able to map one domain onto another only
by means of the underlying language platform, especially the type-system.

There more reasons for it:

- **Stability**: The character or meaning of domain entities is more stable than the structure
   of individual entities. It follows that if the meaning and character is expressed
   by types and not by state, then possible structural changes in the domain model
   should not affect the mapping schema.
- **Type safety**: The type system verifies the consistency of the mapping rules,
   i.e. that the types involved in the mapping can be mapped and composed as intended.
   It can detect missing parts and makes the mapping schema robust against changes
   in the domain models.
- **Early error discovery**: If a static language is used then the consistency
   check and the verifications are carried out at compile-time. It follows that
   possible errors and inconsistencies in the mapping may be caught early.

Java's type system does not provide sufficient means to express the real character
of objects by type. Therefore, designers must resort to the modeling of the object
character by state (e.g. delegation) whereby they incorporate object schizophrenia
into the model.

The concept of traits as implemented in Scala or Groovy fits very well to
the needs of such a mapper.

However, using traits to express multidimensional objects leads to the explosion
of class declarations and boilerplate code.

This problem can be resolved by extending the type system so that the multidimensional
space of all possible object forms can be grasped by a special type expression.

The result of a mapping is a new target domain object composed of the source domains
objects. In order to avoid object schizophrenia the source objects must be
deprived of their identity in behalf of the new target domain object.

It is practically impossible to carry out the identity deprivation in the
current statically typed languages. Therefore an alternative object instantiation
procedure, which generalizes the current one, must be incorporated to the
used language platform.

#####Source Code

The following [link](https://github.com/zslajchrt/morpheus-tutor/tree/master/src/main/scala/org/cloudio/morpheus/scanner)
refers to a source code containing a runnable implementation of the Scaner protodata
case study developed by means [Morpheus](https://github.com/zslajchrt/morpheus).
