---
layout: post
title: "All I want for Christmas is to sign in with Apple"
summary: "TL;DR: Apple please don't make me pay 100€ per year to implement Sign in with Apple on my website. Please make it free and encourage as many people as possible to use it outside the AppleOS environments. I want to implement this as a developer and I want to use the feature as a user."
date: 2019-12-22 12:00:18
comments: false
author: Viorel
category: techy
slug: all-i-want-for-christmas-is-sign-in-with-apple
tags:
- developments
---

**TL;DR: Apple please don't make me pay 100€ per year to implement Sign in with Apple on my website. Please make it free and encourage as many people as possible to use it outside the AppleOS environments. I want to implement this as a developer and I want to use the feature as a user.**

I'm indie-writing a webapp at the moment and I was contemplating about ways to make it easier for my users to sign in. I'd have the normal e-mail/password login and I'd have social logins. I want to make it as easy as possible for my users to board on, so I think the social logins are really a must. 

But the problem is that there are a gazillion "Sign in with-" services and I have to narrow it down. And I sat down and did just that: I narrowed it down to just Google and Apple. It makes sense if you think about it this way: almost all users that will visit my page will own either an Android or an Apple phone. "Sign in with Google" and "Sign in with Apple" will cover most users.

I really really really want to give "Sign in with Apple" to my users but I can't (ok, *I won't*, but hear me out).

### Sign in with Apple

The Apple sign in came a few months ago along with iOS 13 and it's in my top 5 technical novelties this year, thanks to the fact that it's privacy focused:

1. Apple won't track users inside the apps they log in to
2. (and, my absolute favorite part: ) Users can choose not to share their actual e-mail address with the app with the "Hide my e-mail" functionality

I want to talk a little bit about #2: when logging in to a new app, users can choose no to share their e-mail address with that app. Apple then creates a random e-mail address (unique to the app) and does a private relaying of the communication between the app and you - in plain English, the app sends its e-mails to that unique address and Apple forwards the e-mail to your actual e-mail address. The best part? If the app ~~sells~~ loses your e-mail address in a breach, and you start getting spam, you just uncheck a checkbox and you won't receive any more e-mails through that address. And all that already built-in with your Apple account.

*In pragmatic terms, I also believe using the Apple login would help increase my sign-in rate, because oh-no-i-have-to-give-another-app-my-email has just become cost-free. I think this will become more and more important as we become more data-aware.* 

### Then just do it, man!

I actually started implementing it. The first step was "Create a developer account" and the second "Create a new app". The first step I did without paying Apple anything, but, for the second one, I needed to register for the Apple Development program - yearly cost: 100€**.** 

**I am just not willing to pay that at the moment. I don't plan to have a mobile app and I don't want to pile up costs at this stage of the project.**

And frankly, if you're put before the decision of implementing Sign-in with Facebook for free vs Sign-in with Apple for 100€, which one are most likely to choose?

While I wholeheartedly agree with the fee for publishing apps in the App Store, I truly believe that "Sign in with Apple" should be excluded from it. 

**I think making it free and encouraging people to use will have a positive impact on the whole web** - Apple has boasted having 1.4 billion active devices, out of which 900 millions are iPhones. If they manage to convert just 5% percent of their user base to always use Apple login, I believe the increase in privacy would be huge. It would also help educate people to be more careful with their online identities.

I think opening this feature to everybody will bring attention to the new privacy-oriented direction Apple has adopted and it will help Apple have a head-start in the race. I don't doubt many other companies will follow when they see it is want users want.

It's a win-win-win* situation. Apple, please, let's make it work.

*unless, of course, you're someone building an app intending to *lose* the e-mail addresses at some point