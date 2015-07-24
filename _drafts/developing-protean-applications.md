---
layout: post
title: Developing Protean Application - Part 1
comments: true
permalink: developing-protean-applications-part1
---

##Developing Protean Applications - Part 1

Having a company, which serves millions of users and is still on rise, is a dream
of many. Moreover, if the company develops a software product, which is used world-wide,
most likely it is getting a gift in the form of personal details of its users or
their activity. Sooner or later the company's managers will inevitably be asking themselves:
*How could we utilize the large amount of data we have been receiving from our clients for years?*
The most likely answer will be, however, that they believe there must be something
important in their data, **unfortunately they do not know what it is**.

Actually, this gift can easily become both blessing and curse at the same time.

To illustrate the point I am going to present a hypothetical company.

The Company started more than a decade ago developing a search engine, which
quickly became very popular. In the beginning the search engine algorithm was
neutral in terms of the processing of the search query. In other words it only
analyzed the keywords and searched for the best matching pages. Then the Company
realized that the results should reflect the user. To put it bluntly, they
wanted to scan all its users to obtain as much their personal data as possible
and to take it into account in the algorithm.

Because the Company had almost no clue who its users are in the beginning,
it offered users to sign up in the search engine under the pretext that they need
to improve search results.

- At this point the user entity consists of a few simple attributes like ID, email, name...
  However the db is populated with hundreds of millions of entities
- The company starts asking users for private details like the phone number or address
  under the pretext that they need it to provide users with premium services. The entity contains the private info section.
- The company starts tracking the activity of users, the entity is extended by hobbies for instance
- The company unleashes its bots to retrieve additional information about its users from other sites like LinkedIn.
  The entity now contains professional career data.
- The company offers registered users with a free email service. The entity contains contacts.
- Since the company keeps track who is online and knows the contacts of many of its users it can
  implement a chat service.

There are a couple of explicit personal details and relationships between users,
like the network of colleagues or contacts, friends etc. Although important,
such obvious relationships will hardly become the company's market differentiator
since there are many specialized social networks around already.

However, there should exist many other subtle and implicit personal data and
relationships that are not evident at first glance.

For instance, such non-trivial personal data could be age, religion, politics,
sexual orientation, education, health, wage group, important life events like
marriage, child birth, lottery prize, accidents, death and so on. We anticipate
that such data could be extracted from personal resources like clickstream,
keywords, geographical positions, emails, chat rooms etc.

As examples of non-trivial social relationships and interactions can be mentioned:
family, relatives, lawyer<->client, physician<->patient, teacher<->student,
creditor<->debtor, aversions, membership in various institutions and boards,
attending the same cultural or other events. We can assume that these social
interactions could primarily be spotted in and retrieved from the data produced
by the services like email or chat. Of course, the clickstream and keywords
could be helpful as well.

The company believes that having such types of information about its users and their interactions
would make it different on the market. But, there is a big question mark. The managers
must again ask themselves: **Does our data contain such information at all? And if so, how reliable would it be?**

It would be foolish to begin with musing about business models, designing user
interfaces and planing milestones and releases. At this stage it would be nothing
else than building castles in the air.

We have to abandon quickly this top-down approach and start thinking in a **bottom-up**
way, in other words to begin with what we have - plentiful and abundant data.

The precondition of such an approach is that we have already been accumulating
as much varied data as possible for some time and the amount of the data is significant
enough to start playing and experimenting with it.

In the following paragraphs I will be dealing with rather technical aspects of
the development of such applications, which I call **protean**.

###So What Protean Means?

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

In such a case it does not make much sense to try to describe such a data
structure by means of any schema, no matter how bushy the schema would be.
Such an entity is naturally resistant to an all-embracing description.

Previously, a schema usually came into existence in the framework of a detailed data
analysis process, i.e. its existence **preceded** that of the data. The resulting schema
was subject to a very careful change management and the change rate tended to be very low
because everybody was afraid of it. To wrap up, **the semantics and the syntax precedes the data**,
which is a consequence of the **top-down** approach of the development.

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
which their data yields a specific kind of information. The schema also establishes
relationships between entities in the context forming so a network of friends,
colleagues or beer lovers, for instance.

Above that, the same entity may be present in different contexts, thus, its meaning
depends on the interpretation of the subject that understands the context's domain.

We may conclude that in this scenario **the data precedes a syntax and a semantics**,
which is a consequence of the **bottom-up** approach.

From what has been described above we can see that such data varies in three
aspects making it *protean*: shapes/content, forms/format, meanings/contextual
or subjective interpretation (semantics).

*Note: Relational tables can be viewed as a special kind of context template.
It is as if a relational table were just a view to a set of entities in our proto-data,
which follow the rules given by the table schema. In theory we could build a universal
RDBMS system working within various contexts identified in a NoSQL database. Leaving
aside possible practical problems with performance, distribution etc., of course.*

###Protean Behavior

Having started storing extremely protean data it is still too early to begin to develop
some meaningful application. Not to mention to develop something what had emerged in
the heads of "analysts" long before we actually started receiving the stream of data.
Such an approach would clearly lead to a waste of time and resources. The typical
reasoning at that time is: We believe that our huge data must be full of gems.
The problem is that we don't know these gems nor how to retrieve them.

It does not mean that we should not muse about potential uses, but we should
be prepared to abandon them quickly when new insights, dug from the data, reach us.

What we should do, however, is to examine and wring the proto-data as much as
possible with the goal to discern any glimpse of some "alliances" of entities that
could be described by some schema and be given some meaning. Such a thing can be
a fetus of a new insight, which can drive the further development of an application
utilizing the insight.

Once a well-defined context is available we can build an application for it.
As indicated above, the context consists of a network of interconnected entities,
which follow the context rules (schema). The context also defines the semantics of
such a network.

To conclude, we do not build applications directly on top of proto-data.
Instead, firstly we identify subsets of data entities having something in common,
then we describe it by means of a schema, give it some meaning and establish relationships
between the entities - a network. The result is the so-called context, which is
the foundation on top of which we can build some applications.

###Technologies

The tools that we need to build protean systems and applications can be grouped
to four groups reflecting the stages of the development.

  1. Persistence
  2. Analytics tools to identify contexts
  3. Context management, network builders
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
like Hadoop, Spark, Impala and many others may be used to identify contexts in
proto-data.

The output of the context analysis is a description of the new context. The description
should allow a straightforward introduction of the new context into the system.

This task may require an experienced data science team, which is supposed to uncover
numerous categories of complex dynamics of interactions among entities from noisy proto-data.

####Context management

The context management basically solves these tasks:
  1. Introducing new contexts into the system
  2. Tracking entities in proto-data and assigning them to existing contexts (online or offline tagging)
  3. Indexing
  4. Building networks on top of the entities in a context (relationship management)
  5. Mediating communication between applications and contexts (connection management, queries etc.)

These tasks may remind a database engine, which de-facto is in a broader sense of the term.

At the time of writing this article I am not aware of any technology that would
address these requirements optimally.

####Development tools

Developing applications on top of contexts should be much easier than doing
the same thing on top of raw proto-data. The reason is that contexts served
by the context management contain a lot of functionality that would have to be
implemented by the application otherwise. Also the contexts should be as much
domain-specific as possible.

As we will see in the next part of this article, even in a simple application
built on top a simple network the same entity may assume various roles in dependence
on the other role with which it has the relationship.

Dynamic languages like Python, Ruby or JavaScript may sound like good candidates,
however, the refactoring of protean application written in these languages would become
extremely error-prone. It is pretty easy to forget to change, add or remove
something somewhere while something new is being added to the application since
the network of mutual relationships between entities can be very complicated.

##What's Next?
In the next part of this article I will not be dealing with the technology stack
needed for the persistence of proto-data nor with analytics tools for context identification.
Instead I am focusing on building networks and applications on top of a database of
users of a fictional popular web application.

The goal is to show how with Scala and Morpheus we can build protean applications
while preserving all benefits associated with Scala's strong
static type systes along with additional type-safe metamorphism of objects
provided by Morpheus.
