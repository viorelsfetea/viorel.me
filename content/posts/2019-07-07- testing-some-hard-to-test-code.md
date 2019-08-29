---
layout: post
title: 'Easily testing some seemingly hard to test code'
summary: Working on the "find the middle point of a polyline" task, I hit a wall in terms of unit testing my code. This wasn't like most of my other tasks - I just couldn't properly visualize the GivenWhenThen of it. How am I supposed to accurately unit test this thing out then?
date: 2019-07-07 12:00:18
comments: false
category: techy
author: Viorel
slug: testing-some-hard-to-test-code
---
Working on the "[find the middle point of a polyline](https://viorel.me/2019/find-the-middle-centroid-of-a-polyline/)" task, I hit a wall in terms of unit testing my code. This wasn't like most of my other tasks - I just couldn't properly visualize the GivenWhenThen of it. How am I supposed to accurately unit test this thing out then? Apart from the tests for the edge cases, which were pretty clear, I had some problems with testing the actual implementation:

1. How do I decide on a list of accurate input data sets (polylines). 
2. How do I know I've set the expected values correctly for the sets?
3. Results are coordinates, more exactly floating point numbers. Do I assert with tolerance? How much should the tolerance be?

In regards to #1, building a comprehensive group of input polylines was easy. Confidently getting the results I needed, not so much. And I had two options: one was to implement the algorithm, draw the polylines and the results (middle points) on a map, see if they *looked* ok and use them. The other one was to load the polylines in an external tool or use an external implementation of the same or a similar algorithm to calculate what I need and then use those values. I discarded the first option quickly because my eye-to-pixel-to-meter coordination is awful and, for the second, I couldn't find what I needed.

After some vigorous searching and talking to some people, I realized I was approaching the problem too rigidly. 

**The solution** was renouncing calculating the coordinates to their last decimals for the unit tests, but rather just prove that 1) the point is *on* the polyline and 2) the distance to the point equals the distance to the middle of the polyline. So, not check the actual results, but checking the properties of the results. 

In the end, it was simple enough, I just needed to load the polylines in an external tool that measures total distances (I just used the Measure Distance feature from Google to get the distance in meters, cut it to half and use that as hardcoded reference inside the code). Seeing if the point is on the polyline was as easy as seeing if the point is on the line between the point before the center and the point after the center

**Showerthought: Could I have TDD it?**

I didn't TDD this code when I wrote it simply because I didn't think it was possible. It happens sometimes that I stop and stare at a problem and discard finding a solution because it seems impossible. Most of the time, I just need to think out of the box, or stop thinking outside the box. 

This was, I think, a case of too much thinking outside the box. As I said, one of my first thoughts was drawing polylines on the map and making sure I get the middle point (somehow, visually perhaps) but the point precision I needed to achieve would have made this impossible. I thought of getting straight polylines, polylines that are exactly 1km long, draw polylines at the Equator. *Ooooh man.*

After I've split the problem into small the small steps and I decided what approach I was going to take, it should've been pretty clear: in order to go the TDD way, and be sure that the results are what I want, I needed to apply a formula that is *proven* to correctly calculate what I needed and have a set of results calculated by the said formula. And how do you prove a mathematical formula? Yup, mathematical induction. Simple, plain old mathematical induction.

**TDD based on mathematical induction**

TDD and induction are somehow opposite concepts. The first implies writing the result before writing the formula and the other one implies using the formula to calculate two results, a base case and an inductive step. But why not combine the concepts and write two unit tests on the code: one for the base case and one for the inductive step. To get the proper results for the tests, I can use a separate method that I know for sure will give me the correct results: I can draw the polylines on some squared paper and count the squares.

1. The base case should be the simplest polyline I can draw. A polyline made of two points is a crappy polyline, but a polyline nonetheless. So I can choose a polyline where P1 was {0, 0} and P2 was {2, 2}. I could've picked {1, 1} for P2, but I want to work with whole numbers. For this, it doesn't take a mathematical genius (which I am not, by far) to figure out that the coordinate of the middle point is {1, 1}. I basically have to count to one.
![](/assets/images/blog/Untitled-f1096572-271b-454f-81aa-20e0b6919d20.jpg)

1. The inductive step: add another line to the polyline, so it's now formed by 3 points - P1{0, 0}, P2{2, 2} and P3{4, 4}. This was a little bit trickier than the base case, because I have to count to two, but I can manage nonetheless and I can be pretty confident in the results.
![](/assets/images/blog/Untitled-4a978cdf-9288-4ade-82f5-ff52c95cf004.jpg)

Now, all I would've needed to do was write a formula that would make both tests pass. If both tests are green, that means the formula is proven.
