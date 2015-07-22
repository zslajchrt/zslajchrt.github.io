---
layout: post
title: Developing Protean Application With Morpheus - Part 1
comments: true
permalink: developing-protean-applications-part1
---

###Developing Protean Applications With Morpheus - Part 1

####What Protean Means?

[The Free Dictonary](http://www.thefreedictionary.com/protean) defines *protean*
as follows: *Readily taking on varied shapes, forms, or meanings.*. This
definition fits exactly the character of both data and behavior found in many
today's applications.

Protean Data

Today, as the amount of available information and its sources is growing almost uncontrollably,
it becomes increasingly difficult to manage it by traditional means.

Besides its abundance, the heterogeneity of data also makes its management more difficult.
We often have to keep the data as it is, i.e. heterogeneous, since any normalization would lead to
some loss of information.

It does not make much sense to try to describe such a data structure by means of
any schema, no matter how bushy the schema would be. Such an entity is naturally
resistant to any description.

In extreme case a data entity can be so *protean* that the only attribute, which is
always present, is the entity's identity, while all other attributes or
groups of attributes are more or less arbitrary or specific to a group of entities only.
For example groups of users in a social network.

Such groups of users may evolve unpredictably and the social network provider
may come up with some ideas on the utilization of such groups by developing
specific applications that suit well the needs and expectations of these groups.

There is a number of recently developed storage systems, which can handle peta-bytes of
heterogeneous data, however, often at the expense of the relationships (no joins)
and metadata (no schema) management, among others.

It means, thus, that these missing tasks must be solved somewhere in the application
layers stack. And the lower such a management takes place in the stack the better.
Ideally, it would be the language, or eventually the language platform, which
would take on this responsibility. In the worst case, it would be done by the application
itself. Usually it is done by an intermediary big data platform.

....

Dynamic languages like Python, Ruby or JavaScript ... , but the refactoring of
the application written in these languages becomes extremely error-prone. It is
pretty easy to forget to change, add or remove something somewhere when something
new is being added to the application since the network of mutual
relationships between entities can be extremely complicated.


Protean Behavior


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
