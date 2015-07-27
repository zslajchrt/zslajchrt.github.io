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
In most cases the answer is, however, that they believe there must be something
of extraordinary importance in their data, **unfortunately they do not know what it is**.

Actually, this gift can easily become both blessing and curse at the same time.
To illustrate the point let me present a fictional company.

"A decade ago the Company launches a search engine, which quickly becomes very popular.
In the beginning the search engine algorithm is neutral in terms of the processing
of the search query. In other words it only analyzes the keywords and searches for
the best matching pages. Then the Company realizes that results should reflect
the user too. To put it bluntly, they want to scan all its users to obtain
as much their personal data as possible and to take it into account in the algorithm.

Because the Company has almost no clue who its users are at this moment,
it offers users to sign up to the search engine under the pretext that they need
to improve search results.

Since the users are reluctant to reveal more personal information, the Company
starts to track their activity through a web browser plugin. In addition,
the Company unleashes its bots to retrieve additional information, like career history,
from specialized social networks.

The Company also launches an email service offering virtually unlimited space for
registered users.

Since the company keeps track of who is online and of who knows who thanks to
its email service, it comes up with a chat room seamlessly integrated with
other services.

At this point the Company's database already contains petabytes of personal data
of its users, their email and chat communication plus individual clickstreams
and data gleaned from other social networks.

It is also the moment when the managers ask the above-mentioned question about
some utilization of the data.

Having look at their data they see user records containing a couple of
plain personal details and some basic relationships between users, like networks
of colleagues or contacts, friends etc. Although important, such ordinary information
will hardly become the company's market differentiator since there are many
specialized social networks around already.

They believe, however, that there must exist plenty of subtle and extremely
valuable data and relationships, which are not evident at first glance.

For instance, such non-trivial personal data could be age, religion, politics,
sexual preference, education, health, wage group, important life events like
marriage, child birth, lottery prize, accidents, death and so on. They anticipate
that such data could be extracted from personal resources like clickstream,
keywords, geographical positions, emails, chat rooms etc.

Further, they identify some examples of non-trivial social relationships and interactions
like family, relatives, lawyer<->client, physician<->patient, teacher<->student,
creditor<->debtor, aversions, membership in various institutions and boards,
attending the same cultural or other events. They assume that these social
interactions could primarily be spotted in and retrieved from the data produced
by the services like email or chat. Of course, the clickstream and keywords
could be helpful as well.

The company believes that having such types of information about its users and their interactions
would make it different on the market. But, there is a big question mark. The managers
must again ask themselves: 'Does our data contain such information at all? And if so, how reliable, sensitive and valuable would it really be?'"

It would be foolish to begin with musing about business models, designing user
interfaces and planing milestones and releases at this stage. It would be nothing
else than building castles in the air.

Without a good knowledge of the data it is practically **impossible** to follow
the traditional **top-down** approach of application development, in which **use-cases
define data**. Before going to more technical aspects let me briefly analyze
[the knowledge matrix](https://en.wikipedia.org/wiki/There_are_known_knowns) summarizing
our ability to identify some information versus our ability to asses its value.

1. **We know it is there and we know its value** (*Known knowns*): Precisely identified
information with precisely identified value. As far as the Company's data is concerned,
such information consists of essential user's personal data like name, date of birth,
basic relationships between users etc. The overall value of such information is low,
since it is pretty basic, dull and unexciting.

2. **We know it is there, however, its value is uncertain or precarious** (*Known unknowns*):
The Company identifies a number of instances of such information, for example from
the clickstream or search phrases, such as bank accounts and transactions, sexual
preference, health status and other private and confidential information. Nevertheless,
it must conclude that some pieces of the information have uncertain commercial value
while the others may be too risky, delicate and possibly damaging for the company
if handled inappropriately. The uncertainty is also rooted in moral and
ethical constrains or limitations arising from end-user licenses.

3. **We do not know whether it is there, but if so, we would know its value** (*Unknown knowns*):
The Company's analysts would be glad to get such information from the data, since
it would have an indisputable value. Sadly, its presence is far from obvious
in contrast to the previous cases. Such insights might come, for instance, from the
field of marketing, such as the ability to predict customers behavior, or from
the health field, such as warning users of upcoming possible health deterioration
([bipolar](http://www.ncbi.nlm.nih.gov/pubmed/25827034),
[schizophrenia](https://epianalysis.wordpress.com/2013/04/23/dataminementalhealth/), epilepsy).

4. **There may be something which is out of scope of our imagination now and even
when brought to light, it may not be clear if it is worth of anything** (*Unknown unknowns*) -
A "strange fish", which may be as something potentially really unique and revolutionary
as something useless.

To put it in a nutshell, as long as the Company **can identify** some information,
it is either **unexciting or precarious**, as far ar the commercial value is concerned.

But there are yet the two categories concerning **unidentified information**.
In the former the analysts can enumerate and appreciate such **wishful insights**,
however, the data may or may not yield them. In the latter, the insights, **potentially
groundbreaking**, are shrouded in a veil. The analysts cannot even imagine them.

In contrast to the first two categories of knowledge, the last two may produce
something really **exciting**. The common ground for both is that without
some **intensive research** they will barely produce some value.

As far as the *uknown-known* case is concerned, the research could be rather
**short-term** engaging the majority of the R&D team. The more or less bright contours
of the longing visions should allow the analysts to specify goals and criteria
for abandoning a branch of the research.

Contrarily, the research in the *uknown-unknown* case should be considered
a **long-term** activity with hazy goals, a terra incognita. The lack of imagination
precludes any attempt to specify both goals and criteria for abandoning the research.
Researchers should be highly creative, rather eclectic, combining various methods and tools,
stimulating imagination during brainstorming sessions, seeking inspiration and
parallels in nature, music, literature, history and so on. In any case, this research
will require a small, dedicated team of the most talented guys in the Company.

Let me summarize the key characteristics of the four epistemological categories:

1. Known-Known: too dull and unexciting
2. Known-Unknown: too risky, it's not much about research, rather about business strategy (and the stomach)
3. Unknown-Known: promising, a short-term research, a feasibility study as the first deliverable
4. Unknown-Unknown: a terra incognita, it may or may not yield something groundbreaking,
requiring a long-term research by a small team

In any event, the Company must make the important decision as early as possible,
whether it is willing to invest into the research or not.

If not, the Company had better concentrate on the development of its flag-ship
application and use the data to improve it.

If so, the company should assess the proportions of the 'unknown-known' and 'unknown-unknown'
research teams. Perhaps, applying [Pareto's 80-20 rule](https://en.wikipedia.org/wiki/Pareto_principle)
might be a good starting point. The proportions of effort invested into the individual
knowledge categories might then be adjusted as follows:
Known-Known 0%, Known-Unknown 0-10%, Unknown-Known 80%, Unknown-Unknown 10-20%.
Furthermore, the Company will have to embrace some **bottom-up** approach to the development.

In the following paragraphs I will be dealing with the development of applications,
which emerge as the result of such research. Since developing such applications
is often like walking on quicksand, I call this breed of applications **protean**.

###So What Protean Means?

The word actually refers to Proteus, a god of sea and rivers in Greek mythology,
also called the god of "elusive sea change". [The Free Dictonary](http://www.thefreedictionary.com/protean)
defines *protean* as follows: *Readily taking on varied shapes, forms, or meanings.*. And this
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
Such an entity is naturally resistant to an all-embracing description.

Previously, a schema usually came into existence in the framework of a detailed data
analysis process, i.e. its existence **preceded** that of the data. The resulting schema
was subject to a very careful change management and the change rate tended to be very low
because everybody was afraid of it. To wrap up, **the semantics and the syntax precedes the data**,
which is a consequence of the **top-down** approach of the development.

Contrarily, when collecting large amounts of heterogeneous data, sometimes we have
a very limited knowledge of the data in the beginning. We may call such data
collected in the early stages as *protodata*, since it has a very limited applicability
and only with the arrival of more data we can expect, or hope at least, that some
schema or schemas along with some semantics will emerge.

As an example we can take the above-mentioned groups of users in a social network,
which may emerge unpredictably and the social network provider may
react by coming up with some ideas on the utilization of such groups by developing
specific applications that suit well the needs and expectations of these groups.

The schema in this concept is a pattern or a template used to select groups of
associated entities sharing some qualities rather than a rule dictating
the structure of data. These associations of entities establish *domains* within
which their data yields a specific kind of information. The schema also establishes
relationships between entities in the context forming so a network of friends,
colleagues or beer lovers, for instance.

Above that, the same entity may be present in different domains, thus, its meaning
depends on the interpretation of the subject that understands the domain.

We may conclude that in this scenario **the data precedes a syntax and a semantics**,
which is a consequence of the **bottom-up** approach.

From what has been described above we can see that such data varies in three
aspects making it *protean*: shapes/content, forms/format, meanings/domain-specific
or subjective interpretation (semantics).

*Note: Relational tables can be viewed as a special kind of domain templates.
It is as if a relational table were just a view to a set of entities in our proto-data,
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
be prepared to abandon them quickly when new insights, dug from the data, reach us.

What we should do, however, is to examine and wring the protodata as much as
possible with the goal to discern any glimpse of some "alliances" of entities that
could be described by some schema and be given some meaning. Such a thing can be
a fetus of a new insight, which can drive the further development of an application
utilizing the insight.

Once a well-defined domain is available we can build an application for it.
As indicated above, domains consist of a network of interconnected entities,
which follow the domain rules (schema). The domain also defines the semantics of
such a network.

To conclude, we do not build applications directly on top of protodata.
Instead, firstly we identify domains, i.e. subsets of data entities having
something in common, then we describe it by means of a schema, give it some
meaning and establish relationships between the entities - a network.
The resulting domain is the foundation on top of which we can build some applications.

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
like Hadoop, Spark, Impala and many others may be used to identify domains in
protodata.

The output of the domain analysis is a description of the new domain. The description
should allow a straightforward introduction of the new domain into the system.

This task may require an experienced data science team, which is supposed to uncover
numerous categories of complex dynamics of interactions among entities from noisy protodata.

####Domain management

The domain management basically solves these tasks:
  1. Introducing new domains into the system
  2. Tracking entities in proto-data and assigning them to existing domains (online or offline tagging)
  3. Indexing
  4. Building networks on top of the entities in a domain (relationship management)
  5. Mediating communication between applications and domain (connection management, queries etc.)

These tasks may remind a database engine, which de-facto is in a broader sense of the term.

At the time of writing this article I am not aware of any technology that would
address these requirements optimally.

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
In the next part of this article I will not be dealing with the technology stack
needed for the persistence of proto-data nor with analytics tools for domain identification.
Instead I am focusing on building networks and applications on top of a database of
users of a fictional popular web application.

The goal is to show how with Scala and (Morpheus)[https://github.com/zslajchrt/morpheus]
we can build protean applications while preserving all benefits associated with Scala's strong
static type system along with additional type-safe metamorphism of objects
provided by Morpheus.
