---
layout: post
status: draft
published: false
title: Mongo-Radical - Introducing CBMongoDB, a MongoDB module for Coldbox
description: "The one in which I get all fancy-pants with NoSQL and (kind of) become a convert"
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
- Coldbox

fa_class: "fa-database"
---

I have this application, maybe some of you have one like it too.  I first developed it all the way through public BETA in 2008, but it never took off and I was never really happy with it. In 2010 I took the site offline, but I never really stopped wanting to give it another go.

In retrospect, I realize the problem was, in part, a database problem and, to a larger degree, a user interface problem.  The javascript technology and browser support just wasn't available for the interface I wanted to use and the relational data model was complex and, well, slow.  Really slow.

I dusted off the app this fall - winter up here in Michigan, USA is my time for personal projects - as it seemed like the stars were aligning to make what I wanted to do with it possible.

The old app was developed in [Model-Glue](http://en.wikipedia.org/wiki/Model-Glue), running on MySQL with [Reactor](https://github.com/ReactorORM/reactor) as the ORM framework. It was lightweight and fast, but I've never been a fan of XML configuration files.  More importantly development updates to the database schema and application were cumbersome and feature upgrades were often a tedious process.  I became frustrated with the experience it was producing, the development headaches for myself and, eventually, shelved the project.

Once I dusted it off, my first thought was an in-place upgrade to server-side ORM and Coldbox as the app framework.  I've deployed several large projects using CF ORM, though and, well,  

Enter NoSQL
-----------
NoSQL DBMS's, frankly, kind of scare me.  Not from a technical standpoint, but absence of all but indexing to force schema compliance gives me acid flashbacks to the some of the wild and wooly web application DB's I've had to update.  You know the ones:  No foreign keys, primary keys manually generated at runtime from arcane methodologies, storing the same information in 12 different tables because of #1.





