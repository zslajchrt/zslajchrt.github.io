---
layout: post
title: Developing Protean Application With Morpheus - Part 2
comments: true
permalink: developing-protean-applications-part2
---
###So What Protean Means?

The word actually refers to Proteus, a god of sea and rivers in Greek mythology,
also called the god of "elusive sea change". [The Free Dictonary](http://www.thefreedictionary.com/protean)
defines *protean* as follows: *Readily taking on varied shapes, forms, or meanings*. And this
definition fits exactly the character of both data and behavior found in many
today's big-data applications.

###Protean Data

Today, as the amount of available information and its sources is growing almost
uncontrollably, it becomes increasingly difficult to manage it by traditional means
and practices.

Besides its abundance, the heterogeneity of data is a key factor in the struggle to
harness *big data*. We often have to keep the data as it is, i.e. heterogeneous,
since any normalization would lead to some loss of information, for instance.

In extreme case a data entity can be so *protean* that the only attribute, which is
always present, is the entity's identity, while all other attributes or
groups of attributes are more or less arbitrary or specific to a group of entities only.

In such a case it does not make much sense to try to describe such a data
structure by means of any schema, no matter how bushy the schema would be.
Such an entity is naturally resistant to an all-embracing description, i.e. *metadata*.

Previously, metadata usually came into existence in the framework of a detailed data
analysis process, i.e. its existence **preceded** that of the data. The resulting metadata
was subject to a very careful change management and the change rate tended to be very low
because everybody was afraid of it. To wrap up, **the metadata precedes the data**.

On the other hand, when collecting large amounts of heterogeneous data, sometimes we have
a very limited knowledge of the data in the beginning. We may call such data
collected in the early stages as *protodata*, since it has a very limited applicability
and only with the arrival of more data we can expect, or hope at least, that some
metadata will emerge.

We may conclude that in this situation **the data precedes the metadata**.
Furthermore, there may emerge other alternative metadata describing the same data.

The metadata in this concept can be seen as a pattern or a template used to select groups of
associated entities sharing some qualities rather than a rule dictating
the structure of the data. These associations of entities establish *domains* within
which their data yields a specific kind of information. The metadata also establishes
relationships between entities in the domain forming so a network of friends,
colleagues or beer lovers, for instance.

Above that, the same entity may be present in different domains, thus, its meaning
depends on the interpretation of the subject that understands the domain.

From what has been described above we can see that such data varies in three
aspects making it *protean*: shapes/content, forms/format, meanings/domain-specific
or subjective interpretation (metadata).

*Note: Relational tables can be viewed as a special kind of domain templates.
It is as if a relational table were just a view to a set of entities in our protodata,
which follow the rules given by the table schema. In theory we could build a universal
RDBMS system working within various domains identified in a NoSQL database. Leaving
aside possible practical problems with performance, distribution etc., of course.*

###Protean Behavior

Having started storing extremely protean data it is still too early to begin to develop
some meaningful application. Not to mention to develop something what had emerged in
the heads of analysts long before we actually started receiving the stream of data.
Such an approach would clearly lead to a waste of time and resources. The typical
reasoning at that time is: *We believe that our huge data must be full of gems.
The problem is that we don't know these gems nor how to retrieve them.*

It does not mean that we should not muse about potential uses, but we should
be prepared to abandon them quickly and change the course when new insights,
dug from the data, reach us.

What we should do, however, is to examine and wring the protodata as much as
possible with the goal to discern any glimpse of some "alliances" of entities that
could be described by some schema and be given some meaning. Such a thing can be
a fetus of a new insight, which can drive the further development of an application
utilizing the insight.

Once a well-defined domain is available (i.e. metadata), we can build
an application for it. As indicated above, domains consist of a network of
interconnected entities, which follow the domain rules anchored in the metadata.

To conclude, we do not build applications directly on top of protodata.
Instead, firstly we identify domains, i.e. subsets of data entities having
something in common, then we describe it by means of metadata giving it some
meaning and establishing relationships between the entities (network).
The resulting domain is the foundation on top of which we can build some applications.

###Technologies

The tools that we need to build protean systems and applications can be grouped
to four groups reflecting the stages of the development.

  1. Persistence
  2. Analytics tools to identify domains
  3. Domain management, network builders
  4. Development tools

####Persistence

There is a number of recently developed storage systems like NoSQL databases,
which can handle peta-bytes of heterogeneous data, however, often at the expense
of the relationships (no joins) and metadata (no schema) management, among others.

It means, thus, that these missing tasks must be solved somewhere in the stack of
application layers. And the lower such a management takes place the better.
Ideally, it would be the language, or eventually the language platform, which
would take on this responsibility. In the worst case, it would be done by the application
itself. Usually it is done by an intermediary big data platform.

####Analytics tools

The big-data analytics has advanced pretty far over the past decade. Technologies
like Hadoop, Spark, Impala, Splunk and many others may be used to identify domains in
protodata.

The output of the domain analysis is a description of the new domain. The description
should allow a straightforward introduction of the new domain into the system.

This task may require an experienced data science team, which is supposed to uncover
numerous categories of complex dynamics of interactions among entities from noisy protodata.

####Domain management

The domain management basically solves these tasks:

  1. Introducing new domains (metadata) into the system
  2. Tracking entities in protodata and assigning them to existing domains (online or offline tagging)
  3. Indexing
  4. Building networks on top of the entities in a domain (relationship management)
  5. Mediating the communication between applications and domain (connection management, queries etc.)

These tasks may remind a database engine, which de-facto is in a broader sense of the term.
At the time of writing this article I am not aware of any technology that would
address these requirements optimally.

The domain management should avoid any shuffling or copying of data. There is
always only one instance of a data entity in the protodata, even if
the entity may belong to several domains. These domains may represent the
entity in different shapes so as to make its usage easier for domain applications,
however, under the hood, it is always the same entity instance playing various
roles only.

The domain also *animates* its entities by attaching characteristic behavior to them.
For instance, if the domain consists of pictures, every picture entity would
be enhanced by methods for editing, such as resizing, rotating, applying various effect
filters etc. It is important to emphasize, that this enhancement is restricted to the scope of
the picture domain only. So if an entity belonging to domain `A` belongs also
to domain B, the enhancements of the entity made in `A` are not seen in `B`
and vice versa. And of course, this does not guarantee that changes of the entity's state
made in `A` through its enhancement are not seen in `B`.


####Development tools

Developing applications on top of domains should be much easier than doing
the same thing on top of raw protodata. The reason is that domains served
by the domain management contain a lot of functionality that would have to be
implemented by the application otherwise. Also the domains should be as much
clearly defined as possible.

As we will see in the next part of this article, even in a simple application
built on top a simple network the same entity may assume various roles in dependence
on the other role with which it has the relationship.

Dynamic languages like Python, Ruby or JavaScript may sound like good candidates,
however, the refactoring of protean application written in these languages would become
extremely error-prone. It is pretty easy to forget to change, add or remove
something somewhere while something new is being added to the application since
the network of mutual relationships between entities can be very complicated.

##What's Next?
In the next part I will not be dealing with the technology stack
needed for the persistence of protodata nor with analytics tools for domain identification.
Instead I am focusing on building applications on top of a domain.

The goal is to show how with [Morpheus](https://github.com/zslajchrt/morpheus) and Scala
we can build protean applications while preserving all benefits associated with Scala's strong
static type system along with additional **type-safe metamorphism** of objects
provided by Morpheus.
