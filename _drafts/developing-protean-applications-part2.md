---
layout: post
title: Developing Protean Application With Morpheus - Part 2
comments: true
permalink: developing-protean-applications-part2
---

####Case Study Overview

 - A popular web site like a search engine
 - In the beginning it has no clue about its users
 - Registration of users to improve search results and targeting ads
 - At this point the user entity consists of a few simple attributes like ID, email, name...
   However the db is populated with hundreds of millions of entities
 - The company starts asking users for private details like the phone number or address
   under the pretext that they needed to provide users with premium services. The entity contains the private info section.
 - The company starts tracking the activity of users, the entity is extended by hobbies for instance
 - The company unleashes its bots to retrieve additional information about its users from other sites like LinkedIn.
   The entity now contains professional career data.
 - The company offers registered users with a free email service. The entity contains contacts.
 - Since the company keeps track who is online and knows the contacts of many of its users it can
   implement a chat service.


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
