---
layout: post
title: "Domain Model vs Persistence Model"
modified:
categories: blog
excerpt: ""
tags: [DDD]
image:
  feature:
date: 2016-06-27T15:39:55-04:01
---

## 1) Persistence and Domain model as the same thing
If you are in a new project with a database designed from zero for it I would probablly suggest this option. Yes, the domain and your knowledge about it will change constantly and this will demand refactoring that will affect your database, but I think in most cases it's worth it.

With Entity Framework as your ORM you can **almost** mantain your domain models entirely free of ORM concerns using [fluent mappings][1].

***Good parts:***

* Fast, easy, beautiful (if the database is designed for that problem)

***Bad parts:***

* Maybe the developers starts to think twice before to do a change/refactoring in the domain fearing that it will affect the database. This fear is not good for the domain.
* If the domain starts to diverge too much from the database you will face some difficulties to maintain the domain in harmony with the ORM. The closer to the domain the harder to configure the ORM. The closer to the ORM the dirtier the domain gets.

## 2) Persistence and Domain model as two separated things
It will get you free to do whatever you want with your domain. No fear of refactorings, no limitations provinients from ORM and database. I would recomend this approach for systems that deal with a legacy or bad designed database, something that will probably end messing up your domain.

***Good parts:***

 * Completely free to refactor the domain
 * It'll get easy to dig into another topics of DDD like [Bounded Context][2].

***Bad parts:***

 * More efforts with data conversions between the layers. Development time (maybe also runtime) will get slower.

 * But the principal and, believe me, what will hurt more: *You will lose the main beneffits of using an ORM!* Like [tracking changes][3]. Maybe you will end up using frameworks like [GraphDiff][4] or even abandon ORM's and go to the pure ADO.NET.

> Are there any issues in my approach of mapping the models?

I agree with @jgauffin: *"it's in the repository that the mapping should take place"*. This way your Persistence models will never get out from the Repository layer, in preference no one should see those entities (unless the repository itself).


  [1]: https://msdn.microsoft.com/en-us/data/jj591617.aspx
  [2]: http://martinfowler.com/bliki/BoundedContext.html
  [3]: https://msdn.microsoft.com/library/dd456848(v=vs.100).aspx
  [4]: https://github.com/refactorthis/GraphDiff