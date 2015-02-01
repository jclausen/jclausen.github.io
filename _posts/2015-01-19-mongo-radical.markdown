---
layout: post
status: publish
published: true
title: Mongo-Radical - Introducing CBMongoDB, a MongoDB module for Coldbox
description: "The one in which I dust off an old application, get all fancy-pants, and write my own Coldbox module"
author:
  display_name: Jon Clausen
author_email: jon_clausen@silowebworks.com
author_url: http://jonclausen.com
categories:
- Programming
tags:
- NoSQL
- MongoDB
- Coldfusion
- Lucee
- CFML

fa_class: "fa-database"
---

I have this application, maybe some of you have one like it too.  I first developed it all the way through public beta in 2008, but it never took off and I was never really happy with it. In 2010 I took the site offline, but I never really stopped wanting to give it another go.

In retrospect, I realize the problem was, in part, a database problem and, to a larger degree, a user interface problem.  The javascript technology and browser support just wasn't available for the interface I wanted to use and the relational data model was complex and, well, slow.  Really slow.

I dusted off the app this fall - winter up here in Michigan, USA is my time for personal projects - as it seemed like the stars were aligning to make what I wanted to do with it possible.

The old app was developed in [Model-Glue](http://en.wikipedia.org/wiki/Model-Glue), running on MySQL with [Reactor](https://github.com/ReactorORM/reactor) as the ORM framework. It was lightweight and fast with both of those, but I've never been a fan of XML configuration files and neither are really actively being developed.  Development updates to the database schema and application were cumbersome and feature upgrades were often a tedious process.  I became frustrated with the experience it was producing, the development headaches for myself and, eventually, shelved the project.

Once I dusted it off, my first thought was an in-place upgrade to server-side ORM and Coldbox as the app framework.  I've deployed several large projects using CF ORM, though and, while I like its ease of use, it has a massive footprint for the convenience it brings and can be slow when handling multi-dimensional relationships.  For this app, I wanted nimble, flexible and fast so I decided to look at...  

NoSQL
-----
NoSQL DBMS's, frankly, kind of scare me.  Not from a technical standpoint, but absence of all but indexing to force schema compliance gives me acid flashbacks to the some of the wild and wooly web application DB's I've had to fix or bring up to date.  You know the ones:  No foreign keys, primary keys manually generated at runtime from arcane methodologies, storing the same information in 12 different tables because of #1.

The paradigm shift of NoSQL, is that the application assumes responsibility for schema control and enforcement and, because I try to develop with protection of myself from myself, I decided to develop an active record(-esqe) facade for my database abstraction layer (in this case the excellent [CFMongoDB library](https://github.com/marcesher/cfmongodb). 

CBMongoDB
---------
Say hello to [CBMongoDB](http://www.coldbox.org/forgebox/view/cbmongodb), a module for the Coldbox platform.

My intent is for the module to develop over time to include GEO spatial capabilities[^1], as well as data relationship [normalization and de-normalization](http://docs.mongodb.org/manual/core/data-model-design/) introspection.  In the meantime, I'll keep working on the app and adding functionality to the module as needed.  Code like the wind. -JC

[^1]: This functionality was pushed to the master branch on Jan 24, 2015.







