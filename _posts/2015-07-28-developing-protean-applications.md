---
layout: post
title: Developing Protean Application - Part 1
comments: true
permalink: developing-protean-applications-part1
---

##Developing Protean Applications - Part 1

Having a company, which serves millions of users and is still on the rise, is a dream
of many. Moreover, if the company develops a software product, which is used world-wide,
most likely it is getting a gift in the form of the personal details of its users or
their activity. Sooner or later the company's managers will inevitably ask themselves:
How they can utilize the large amount of data they have been receiving from their customers
for years. There must be something of extraordinary importance in the data,
*unfortunately they do not know what it is*.

Actually, this gift can easily become both a blessing and curse at the same time.
To illustrate this point let me present a fictional company.

The Company launches a search engine, which quickly becomes very popular.
In the beginning the search engine algorithm is neutral in terms of the processing
of the search queries. In other words it only analyzes the keywords and searches for
the best matching pages. Then the Company realizes that the results should reflect
the user too. To put it bluntly, they want to scan all of its users to obtain
as much of their personal data as possible and to take it into account in the algorithm.

Because the Company has almost no clue who its users are at this moment,
it offers users the opportunity to sign up to the search engine under the pretext that they need
to improve the search results.

Since the users are reluctant to reveal more personal information, the Company
starts to track their activity through a web browser plugin. In addition,
the Company unleashes its bots to retrieve additional information, like career history,
from specialized social networks.

The Company also launches an email service offering virtually unlimited space for
registered users.

Since the company keeps track of who is online and of who knows who thanks to
its email service, it comes up with a chat room seamlessly integrated with
other services.

At this point the Company's database already contains petabytes of the personal data
of its users, their email and chat communication plus individual clickstreams
and data gleaned from other social networks.

This is also the moment when the managers ask the above-mentioned question about
some utilization of the data.

Looking at their data they see user records containing a couple of
plain personal details and some basic relationships between users, like networks
of colleagues or contacts, friends etc. Although important, such ordinary information
will hardly become the company's market differentiator since there are many
specialized social networks around already.

They believe, however, that there must exist plenty of subtle and extremely
valuable data and relationships, which are not evident at first glance.

For instance, such non-trivial personal data could be age, religion, politics,
sexual preference, education, health, wage group, important life events like
marriage, child birth, lottery prizes, accidents, death and so on. They anticipate
that such data could be extracted from personal resources like keywords, clickstreams,
geographical positions, emails, chat rooms etc.

Further, they identify some examples of non-trivial social relationships and interactions
like family, relatives, lawyer<->client, physician<->patient, teacher<->student,
creditor<->debtor, aversions, memberships in various institutions and boards,
attendance at the same cultural events. They assume that these social
interactions could primarily be spotted in and retrieved from the data produced
by the services like email or chat. Of course, the clickstream and keywords
could be helpful as well.

The company believes that having such types of information about its users and their interactions
would make it different on the market. But, there is a big question mark. The managers
must again ask themselves: 'Does our data contain such information at all? And if so,
how reliable, sensitive and valuable would it really be?'

It would be foolish to begin with musing about business models, designing user
interfaces and planing milestones and releases at this stage. It would be nothing
else than building castles in the air.

Without good knowledge of the data it is practically **impossible** to follow
the traditional **top-down** approach of application development, in which **use-cases
define data**. Before going on to more technical aspects let me briefly analyze
[the knowledge matrix](https://en.wikipedia.org/wiki/There_are_known_knowns) summarizing
our ability to identify some information versus our ability to asses its value.

1. **We know it is there and we know its value** (*Known knowns*): Precisely identified
information with precisely identified value. As far as the Company's data is concerned,
such information consists of the user's essential personal data such as name, date of birth,
basic relationships between users etc. The overall value of such information is low,
since it is pretty basic, dull and unexciting.

2. **We know it is there, however, its value is uncertain or precarious** (*Known unknowns*):
The Company identifies a number of instances of such information, for example from
the clickstream or search phrases, such as bank accounts and transactions, sexual
preference, health status and other private and confidential information. Nevertheless,
it must conclude that some pieces of the information have uncertain commercial value
while the others may be too risky, delicate or possibly damaging for the company
if handled inappropriately. The uncertainty is also rooted in moral and
ethical constraints or limitations arising from end-user licenses.

3. **We do not know whether it is there, but if so, we would know its value** (*Unknown knowns*):
The Company's analysts would be glad to get such information from the data, since
it would have an indisputable value. Sadly, its presence is far from obvious
in contrast to the previous cases. Such insights might come, for instance, from the
field of marketing, such as the ability to predict customer behavior, or from
the medical field, such as warning users of upcoming possible health issues
([bipolar](http://www.ncbi.nlm.nih.gov/pubmed/25827034),
[schizophrenia](https://epianalysis.wordpress.com/2013/04/23/dataminementalhealth/), epilepsy).

4. **There may be something which is out of scope of our imagination now and even
when brought to light, it may not be clear if it is worth anything** (*Unknown unknowns*) -
A "strange fish", which may turn out to be something really unique and revolutionary
or something utterly useless.

To put it in a nutshell, as long as the Company **can identify** some information,
it is either **unexciting or precarious**, as far as the commercial value is concerned.

But there are still the two categories concerning **unidentified information**.
In the former the analysts can enumerate and appreciate such **wishful insights**,
however, the data may or may not yield them. In the latter, the insights, **potentially
groundbreaking**, are shrouded in a veil. The analysts cannot even imagine them.

In contrast to the first two categories of knowledge, the last two may produce
something really **exciting**. The common ground for both is that without
some **intensive research** they will barely produce any value.

As far as the *uknown-known* case is concerned, the research could be rather
**short-term** engaging the majority of the R&D team. The more or less clearly
defined vision parameters should allow the analysts and managers to specify goals and criteria for abandoning an unfruitful
branch of research.

On the other hand, the research in the *uknown-unknown* case should be considered
a **long-term** activity with hazy goals, terra incognita. An ignorance of the potential beneficial findings
precludes any attempt to specify both goals and criteria for abandoning the research.
Researchers should be highly creative, rather eclectic, combining various methods and tools,
stimulating imagination during brainstorming sessions, seeking inspiration and
parallels in nature, music, literature, history and so on. In any case, this research
requires a small, dedicated team of the most talented members of the Company.

Let me summarize the key characteristics of the four epistemological categories:

1. Known-Known: too dull and unexciting
2. Known-Unknown: too risky, it's not much about research, rather about business strategy (and the stomach)
3. Unknown-Known: promising, short-term research, a feasibility study as the first deliverable
4. Unknown-Unknown: terra incognita, it may or may not yield something groundbreaking,
requires long-term research by a small team

In any event, the Company must make the important decision as early as possible,
whether it is willing to invest into the research or not.

If not, the Company had better concentrate on the development of its flag-ship
application and use the data to improve it.

If so, the Company should assess the proportions of the *Unknown-Known* and *Unknown-Unknown*
research teams. Perhaps, applying [Pareto's 80-20 rule](https://en.wikipedia.org/wiki/Pareto_principle)
might be a good starting point. The proportions of effort invested into the individual
knowledge categories might then be adjusted as follows:
Known-Known 0%, Known-Unknown 0-10%, Unknown-Known 80%, Unknown-Unknown 10-20%.
Furthermore, the Company will have to embrace some **bottom-up** approach to the development.

In the next part I will be dealing with the development of applications,
which emerge as the result of such research. Since developing such applications
is often like walking on quicksand, I call this breed of applications **protean**.
