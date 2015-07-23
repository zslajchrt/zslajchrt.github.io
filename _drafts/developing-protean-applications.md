---
layout: post
title: Developing Protean Application With Morpheus - Part 1
comments: true
permalink: developing-protean-applications-part1
---

##Developing Protean Applications With Morpheus - Part 1

###What Protean Means?

The word actually refers to Proteus, a god of sea and rivers in Greek mythology,
also called the god of "elusive sea change". [The Free Dictonary](http://www.thefreedictionary.com/protean)
defines *protean* as follows: *Readily taking on varied shapes, forms, or meanings.*. And this
definition fits exactly the character of both data and behavior found in many
today's applications.

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
For example a group of users in a social network having something in common.

In such a case it does not make much sense to try to describe such a data
structure by means of any schema, no matter how bushy the schema would be.
Such an entity is naturally resistant to an all-embracing description.

Previously, a schema usually came into existence in the framework of a detailed data
analysis process, i.e. its existence **preceded** that of the data. The resulting schema
was subject to a very careful change management and the change rate tended to be very low
because everybody was afraid of it. To wrap up, **the semantics and the syntax precedes the data**.

Contrarily, when collecting large amounts of heterogeneous data, sometimes we have
a very limited knowledge of the data in the beginning. We may call such data
collected in the early stages as *proto-data*, since it has a very limited applicability
and only with the arrival of more data we can expect, or hope at least, that some
schema or schemas along with some semantics will emerge.

As an example we can take the above-mentioned groups of users in a social network
(cohorts), which may emerge unpredictably and the social network provider may
react by coming up with some ideas on the utilization of such groups by developing
specific applications that suit well the needs and expectations of these groups.

The schema in this concept is a pattern or a template used to select groups of
associated entities sharing some qualities rather than a rule dictating
the structure of data. These associations of entities establish contexts within
which their data yields a specific kind of information.

Above that, the same entity may be present in different contexts, thus, its meaning
depends on the interpretation of the subject that understands the context's domain.

We may conclude that in this scenario **the data precedes a syntax and a semantics**.

From what has been described above we can see that such data varies in three
aspects making it *protean*: shapes/content, forms/format, meanings/contextual
or subjective interpretation (semantics).

###Protean Behavior

Having started collecting protean data it is still too early to begin to develop
some meaningful application. Not to mention to develop something what had emerged in
the heads of "analysts" and "managers" long before we actually started gathering
the data. Such an approach would clearly lead to a waste of time and resources.

It does not mean that we should not muse about potential uses, but we should
be prepared to abandon it quickly when new insights, dug from the data, reach us.

What we should do, however, is to examine and wring the proto-data as much as
possible with the goal to discern any glimpse of some "alliances" of entities that
could be described by some schema and be given some meaning. Such a thing can be
a fetus of a new insight, which can drive the further development of an application
utilizing the insight.


Previously, the behavior was codified in use-cases developed during the analysis.
Behavior-first, Schema-first

Data-first, data centric development... Now, it is very difficult to come up with some
reasonable application even in the early stages of the data collection ... until
we can discern some contexts ...

Proto-data -> Contexts (Subjective interpretation of data in associated entites)
-> Context Application

###

There is a number of recently developed storage systems, which can handle peta-bytes of
heterogeneous data, however, often at the expense of the relationships (no joins)
and metadata (no schema) management, among others.

It means, thus, that these missing tasks must be solved somewhere in the stack of
application layers. And the lower such a management takes place the better.
Ideally, it would be the language, or eventually the language platform, which
would take on this responsibility. In the worst case, it would be done by the application
itself. Usually it is done by an intermediary big data platform.

....

Dynamic languages like Python, Ruby or JavaScript ... , but the refactoring of
the application written in these languages becomes extremely error-prone. It is
pretty easy to forget to change, add or remove something somewhere when something
new is being added to the application since the network of mutual
relationships between entities can be extremely complicated.




####Case Study Overview

####Data Description

####Modeling Protean Data

####Instantiating and Loading

####Protean Behavior (Offline/Online)

####Protean Networks

####Networks Mapper

####Colleagues Network

####Friends Network

####Application of Friends Network

####Simultaneous Networks
