---
layout: post
title: Developing Protean Application With Morpheus - Part 2
comments: true
permalink: developing-protean-applications-part2
---
###Protodata

Protodata can be described as a collection of diverse data objects having no immediate
and meaningful commercial use or value.

It comes into existence usually as a by-product of some business activity,
such as journal logs, but it may also be the leftovers from some shut down project or
data acquired as part of some acquisition.

Protodata may also be deliberately gathered for a longer period of time without
any specific vision of its future use besides the owner's hope that such a large
amount of data may become a gold-mine in the future.

Applying the knowledge matrix introduced in the previous chapter, the level of the data owner's
knowledge at this stage would correspond to categories *Known-Knowns* or *Known-Unknowns*,
when the known value is insignificant or there is considerable risk embedded
in the attempt to sell data possibly containing delicate information.

The lack of a usage means that there is **no context** within which the data objects
in the protodata would yield some commercially or otherwise valuable information.
The objects can be treated only as bare wrappers of intrinsic properties (i.e.
properties that an object has of itself, independently of other objects,
including its context. [here](https://en.wikipedia.org/wiki/Intrinsic_and_extrinsic_properties))

In order to utilize the protodata, it must be put into a novel context. It can
be achieved, for instance, by combining the protodata with other, often apparently
unrelated, data or with newly emerged information services.

The owner's effort to find a new context for his protodata can go in two directions:
To try to find

1. something valuable
2. anything valuable

The first way is less difficult, since it evaluates existing contexts against the protodata.
For example, a competitor has developed a successful service using data that is
somehow similar to ours. So we will examine our protodata to see whether it is able to
yield similar, or possibly better, information if combined with other available
data sources. This situation corresponds to the *Unknown-Knowns* epistemological
category, since the owner knows the value of the information he would like to retrieve
from the protodata, however, it is still unknown, if there is such information.

The second way is significantly more difficult, since its goal is to
discover a new and original context, which would become a market differentiator.
This approach usually requires a longer research period, while the results are highly uncertain.
The knowledge category in this case corresponds to *Unknown-Unknowns*, since
not only does the owner not know if there is anything especially interesting, but
even if there were anything interesting he would not know its value.

In any case, if some vital context is found it also comes with a new domain model.
Although protodata may be accompanied by some metadata, i.e. having its own
domain model - *proto-domain*, the context domain model may be so structurally
and semantically different from the proto-domain that a direct mapping from
the proto-domain to the context domain can be difficult.

If such a situation occurs, the usual solution is to transform and normalize
the protodata in such a way that the mapping will be easier. However, this
approach has several drawbacks.

First, transformations and normalizations are lossy processes by its nature.
It follows that during such processes some information, which could be potentially
used in future development of the application, will be inevitably lost. In
such a case, the processes will have to be redesigned to provide the required
additional data and the protodata will have to be re-processed. And this can, of course,
consume a lot of resources and time.

Second, if the protodata is just a stream of events, which must be processed in realtime,
then any additional pre-processing could be a source of undesired delays.

Third, when another useful context is found, new transforming routes will have
to be established, which will put an additional burden on the infrastructure.
Moreover, reusing the processes already established for the existing contexts,
may not be possible because of the diverse nature of contexts and their domain models.

Fourth, in the course of time when more contexts are implemented the system of
processes will tend to become unmaintainable and resistant to refactoring.

It seems that the only way to bypass the above-mentioned issues is to try to
**map directly** the context domain onto the proto-domain, regardless of the diverse character
of the two domains, and avoid any intermediary processing.

Furthermore, since the type systems of advanced programming languages are
powerful enough to grasp the complex nature of domain objects, domain models
should model the objects by type as much as possible and not by state.

If domain objects are modeled by state, then the objects' character,
i.e. what the objects are, is retrieved from the properties of the objects. For
example in Java, more complex objects with multidimensional character must
be modeled by means of delegation, which is de-facto a modeling by state, as shown
in the next paragraph.

If domain objects are modeled by type, then the objects' character is retrieved
from their type. This approach has many benefits, since a lot of responsibilities
can be delegated to the type system. Additionally, if the language is static,
many possible errors can be discovered at compile-time.

Languages like Scala or Groovy come with the concept of trait, which
is very useful to model diverse nature of complex objects by type.

The main goal of this work (Morpheus) is to prove that it is possible to construct
a domain mapper as an extension of the language platform. In other words, the mapper
becomes integrated with the language itself in contrast to other mapping tools
([list of object-object mappers](http://stackoverflow.com/questions/1432764/any-tool-for-java-object-to-object-mapping)),
which are built on top of the language. It will also be shown that such a mapper
inherits the nice properties of static and statically typed languages such as
type-safety and early discovery of errors, i.e. the mapping schemas are validated
at compile-time.

In the following paragraph I will present an example on which I would like
to illustrate the described problems as well as to sketch their solution.
The protodata in the example is data collected from a fictional airport scanner,
which is able to recognize objects in baggage.
