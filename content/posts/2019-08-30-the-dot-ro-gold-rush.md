---
layout: post
title: The ".ro" Gold Rush
summary: "In a gold rush, you can mine for gold or you can sell pickaxes"
date: 2018-08-30 12:00:18
comments: false
category: techy
author: Viorel
slug: the-dot-ro-gold-rush
tags:
- bugs
---

<small>_--- So, this reached the [HackerNews front-page](https://news.ycombinator.com/item?id=19017138). What a trill! It was quite exciting to see the sheer amount of traffic that can generate (~35k users in 12 hours). Time to cross that of the bucket list._</small>

_"In a gold rush, you can mine for gold or you can sell pickaxes"_

Back in the day, the Romanian domains registrar (RoTLD) used to sell .ro domains *for life* and a domain used to cost between 30 and 50EUR. Because that's not quite sustainable (hindsight is 20/20), they decided to switch all the domains they already sold *for life* to yearly subscriptions.

The new price is about 6EUR per year (more about that below). They were as fair as they could get about this and considered you prepaid for 5 years when you bought your domain. For domains older than 5 years, you either start paying up or you lose them. Then it’s first-come/first-served to buy the domains again.

Last night (29th of august 2018), a really good batch of .ro domains expired (going from `PendingDeletion` to `Available`). There were *lots* of great domains, but anybody who wanted to buy one was surprised to see the domains went directly from `PendingDeletion` to `Reserved`. One can still buy these domains, but only from auction websites. Where, for example, [http://radio.ro](http://radio.ro/) sold for 330EUR. The Romanian language equivalent of [courses.ro](http://courses.ro/) is at 520EUR with about a month left of the auction.

Domain auction is, of course, nothing new or unusual. What's unusual is how the released domains jumped directly from `PendingDeletion` to `Reserved`. Being Romania, one might suspect corruption. Maybe it is, but probably not.

It actually goes like this: RoTLD really hates selling domains by itself and prefers working through resellers. RoTLD asks 12EUR per year for a new domain if you buy directly from them, but they charge resellers 6EUR. So the resellers charge 9EUR per year for new clients and everybody's happy. Most important, there is no cap in the contract for the maximum price the resellers can charge. I can confirm this because I read the whole 3 pages of said contract.

To become a reseller, you have to pay RoTLD in advance the equivalent of 500 domain-years (3000EUR) and agree to take over all the liabilities regarding the domain. Then, you receive an API through which you register domains automatically.

So, now having that insight, what happened was: resellers, having access to the (uncapped) API, have created scripts that queried continuously to see when a domain is released and, then, immediately reserve it. Once reserved, they can pay in 24 hours or it gets released again.

Now, the best part: some resellers start auctioning the domains *one month before* they get released. They tell you that you can bid on it up until an hour before it should be released. Then, they will try to reserve it and, if they succeed, the highest bidder will be charged. Successful domains are sold for crazy profits. Unsuccessful ones are simply not paid. Usually, you pay recurrently the same amount you initially paid for a domain.

So, probably not corruption. Stupid, yes. Unfair, also yes. Illegal, no. It is basically first/come/first-served only if you can cough up upfront 3000EUR *and* have some coding skills. Otherwise, regular Joes never had a chance of buying the domains. It turned into a race of who has the most money gets the domain they want. If you want a really good domain, you can end up paying 500EUR *per year* for it.

In this case, we, of course, should not forget about Hanlon’s razor.
