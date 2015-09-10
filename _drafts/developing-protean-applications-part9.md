---
layout: post
title: Developing Protean Application With Morpheus - Part 9
comments: true
permalink: developing-protean-applications-part9
---

###Case Study Conclusion

Strong type-systems capabilities cannot be fully utilized when using composition
the platform limitations such as a lack of dynamic features. The type system is
of objects instead. A typical application hoovers somewhere between the property-
The two case studies analyzed in this chapter showed that modeling multidimensional
objects may be surprisingly difficult in Java, Scala and Groovy as three representatives of
current popular languages. Java as a non-trait language has proven to be the least
suitable language for the multidimensional modeling. Because of the lack of traits,
one must resort to compositions and delegations, which leads to obscuring both type
and behavior during consecutive mappings between domains. In Scala each form
of a given object must be declared as a class. Since the number of forms grows
exponentially with the number of dimensions, the number of class declarations
quickly becomes unsustainable. Groovy can cope with this problem by means of
the dynamic traits applied on objects at runtime, however in contrast to Scala,
its weak type system is not able to guarantee the consistency of trait compositions
made in a step-by-step way (i.e. imperatively).

In the course of the case study analysis a new concept of the class builder
has been developed and treated as an additional platform evaluated in each scenario.
The class builder is conceived as an extension of Scala and its purpose is
to generate a class for any possible trait composition given by a model, which
is defined by means of a special type expression. The compiler extension
should be able to check the consistency of the model by decomposing the model
type and examining the dependencies of all individual traits specified in the traits'
self-type.

When instantiating a new object the class builder selects the right trait composition,
called alternative, according to the recommendation of the class builder strategy.
Since the strategy does not determine the trait compositions directly and only hints,
which prefabricated alternatives match some external conditions, it is guaranteed
that the builder always generates a class made up of mutually consistent set of traits
corresponding to one alternative given by the statically checked model.

In case the class builder is instantiated with an incomplete model (i.e. it has
some missing dependencies), a special reference object must be passed to the class
builder's constructor. Through this reference the compiler extension can trace
the model type associated with the reference and verify whether that model
delivers all missing dependencies.

The combination of an incomplete model with the special reference may be considered
an equivalent to the mapping between two domains. Such a mapping has all the appealing
properties formulated in the analysis for domain mappings: it uses neither delegation
nor composition (no type schizophrenia); the source trait proxies form the
resulting target composition in line with the target trait proxies (the source
proxies do not sink).







































```scala

```


```scala

```




```
```


```
```


```
```


```scala
```


```scala

}
```



```scala
```


```scala
```
