---
layout: post
title: "On continuous delivery"
date: 2013-04-30 22:40
comments: false
published: false
categories: [continuous delivery]
---

In our current project, we have managed to reach a steady state of delivering working software into production every six weeks. We reached this state over the course of last 3 years. Now, the release activities have become more mature and well understood and there is no drama around that any more.
<!-- more -->
##How did we reach here?

Automation is the key factor for us to reach this position. We have reached a stage where we can 

##What are the future directions?

##Summary


I've been reading about how the giants on the internet (read Google, Facebook etc) deliver software continuously in order to gain valuable feedback from consumers quickly. Compare that with a typical enterprise software development that might run into months if not years. And towards the end of the development cycle, everybody would be dreading the time the software is supposed to go live. Numerous integration, environment configuration and other issues raise their ugly head further dampening spirits and leading to a death march.
<!-- more -->
In our current project, we've managed to reach a steady state of delivering software into production every six weeks. First a bit of background. The domain is retail and the platform is custom built. There is a web application the end user can use to purchase an item. The web site has also been white labelled for different other vendors who use the same platform to retail their goods.

We reached this stable state over the course of time but we've reached a point where we would still like to release even more frequently but are unable to owing to several factors. The first and foremost being the size of the deployment unit. I believe this alone has the potential to drag your release cycle if it is not kept in check. By size of deployment unit, I mean, the set of binaries/scripts etc that form the deployment package. In our case, the deployment package was comprised of three major parts - the website, the back-end services that support the website and the database changes that are required for the release.



Lets assume we are a team developing the Microsoft Windows operating system. Barring the major releases like Windows 7, Windows 8 etc., we do not expect end users to reformat their disk and install the new operating system. Any fixes, enhancements are delivered via Windows update as Service packs or the likes. 