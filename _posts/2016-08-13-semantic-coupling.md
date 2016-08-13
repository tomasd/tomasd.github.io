---
layout: post
title: The Perils of Semantic Coupling by Michael Nygard
description: "My notes"
category:
tags: [distributed-system]
---

{{ page.title }}
================

<p class="meta">13 Aug 2016</p>

Michael Nygard wrote good article on system coupling some time ago http://www.michaelnygard.com/blog/2015/04/the-perils-of-semantic-coupling/. I've found
it on link aggregator with reference to new clojure.spec library. It put's it in
the new light of distributed data.

 According to Michael you should put only relevant data into the master data system. It's
  a bounded context in DDD methodology. Then in downstream systems you can enrich
  these data:

> We should treat identifiers from other systems as opaque tokens that we map onto our own system’s space.

Using clojure.spec and namespaced keywords you can even describe system data. E.g.
each system/service should have own namespace. Then downstream components like online store
can describe their data in the meaning of upstream components.

Using Michael's example of online retailer, online store can have this description
of the shop item:

```clojure
(s/def :online-store/item
  (s/keys :req [:sku/id
                :pricing/price
                :pricing/VAT
                :ratings/reviews
                :cms/title
                :cms/description
                ...
  ]))
```
