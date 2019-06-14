---
layout: post
title: "A tale of two markers"
summary: "That should have pointed to each other. But didn't."
date: 2018-08-12 16:39:18
comments: false
author: Viorel
category: techy
slug: a-tale-of-two-markers
tags:
- bugs
---
*That should have pointed to each other.* But didn't.

It's a rainy afternoon, I'm seeping from a cup of coffee while scanning the news. There's an article about the earth being flat. I scoff... if only. Wouldn't my life be so much easier right about now?

Today I need to be working on an apparently simple problem: make two pointy markers... uhm... point to each other on a map. I'm supervising an outsourced project and the team had had some problems with getting the angle formula just right and they didn't have the experience and time to investigate further. They asked if we're OK with a hack at this point and if they can carry on with other stuff. "How hacky is this hack?", I ask, right before getting a code snippet containing a single method called *magicAngleAdjustment*. I check the method and assert that the name was, indeed, quite accurately chosen.

![](/assets/images/blog/Untitled-8de188fa-3578-404c-b4fa-25bf37369f15.png)
<span class="small text-center text-block">
*where angle is the result of the atan2 function (in radians) on dot1 and dot2 and lon translates to X on a X/Y plane whereas lat is Y...* Sorry, don't read that again, it's not that important at this particular moment in time.
</span>

Now, being the life of the party that I am, I threw my hand immediately into the air and asked not in a low voice HOW DOES THE MAGIC WORK, THAT'S NOT MAGIC, ISN'T IT? It's not.

## Feet on the ground

So what should have happened? The application is building a track out of points added by a user on the map. Imagine you're planning on going on a hike and you're plotting the route you'll be following. Each point is represented by a pointy-arrow marker and any previous point's tip needs to be rotated to point to the next one. The points themselves are connected by a line:

![](/assets/images/blog/Untitled-a451efcd-a17a-46a8-97e9-9789c15b8578.png)
<span class="small text-center text-block">
The final result
</span>

**So why was this not uncomplicated?** Well, the code was calculating the angle between two points with atan2(firstPoint{X,Y}, secondPoint{X,Y}) and then *magically* (really) adjusting the number because it was always a few degrees off. And there's just one un-magical problem with that: the atan2 function works with some X/Y coordinates on a X/Y plane - a flat plane that is. *And flat the Earth is not.*

Also, as anyone who has been paying just a little bit of attention in school would tell you (not me, I learned this the hard way later), you tend to get pretty silly results when you apply 2D formulas for stuff that's on a 3D plane. Like, let's say, a sphere.

**Why not keep it like that? If it works and all?**

It actually didn't fully work. It sort of worked because our maps are flattened using the MERCATOR projection and this falls right on the projection's biggest advantage: being able to represent great circle lines between some coordinates as straight lines. Take the points farther apart and you'd have to tweak the angle adjustments a little bit. And then a little bit more. Oh and it would have probably not worked at all latitudes larger than 70 degrees. Not that anyone would've used our tool there, but then again, all the good programming books say to never assume what your users will do.

Let's pretend it would have worked perfectly regardless of the scenario, though. This particular piece of code would have been hard to explain and maintain. You can unit test it, dress it in nice clean code, but no matter what you do, it will never be beautiful. Oh, did I mention that the values were selected by someone rotating a marker on the map until it looked right? The problem with this being that no matter what you did, it would have never been accurate. For me, it looked just a little bit off - beauty is in the eye of the beholder, I guess.

## Let's jump in, then

So I rolled up my sleeves and I got to work. Luck has it, a few weeks prior I had a similar task that was heavily relying on getting the difference in degrees between a vehicle's bearing and a road's. I would just need to copy the mathematical formula and I'm done. I was so sure about this working, that I even told the other developer that I will give him the solution in 5 minutes. I guess you know what happens when you make too many assumptions.

**Step zero: copy paste and be done with it**

I took said mathematical formula and I applied it. *The results were even worse than at the beginning and the angles were off by about 100 degrees.* OK, how could that be? I've used the method successfully, I have **unit tested** it. I have manually checked the expected values in the unit test with a protractor. And still. I sighed, this wasn't going to be easy after all. Time to dig in:

**First step: isolate the problem**

First, a little bit about how the markers are drawn and rotated on our maps: the markers are added as DOM nodes, positioned over the map, relative to its top/left corner. On draw/redraw, a marker's geographical coordinates are translated to pixels positions relative to the map's DOM element. If a marker needs to be rotated, a CSS transform is applied on its DOM element.

Replicating the problem outside of the project's code gave me some peace of mind that there was no external influence on how the marker was rotated. That was the good news. The bad news was that the problem was pretty much identical outside of the project. Not it, next step.

**Second step: frantically try to modify the CSS transform property to find a clue. I mean... experiment with the CSS properties**

I suspected at first that CSS might be the culprit, so I tested a few assumptions. One was that the marker was not rotated in the correct direction (maybe instead of rotating it 50 degrees it should rotate -50). Then I checked to see if the rotation somehow has an altered origin: a default origin should be from the center of the element (50%, 50% on its X/Y axis). Changing this would modify the results.

After a few hours of soul-wrenching CSS trial-and-error attempts, I was about to close shop for the day when I noticed something:

![](/assets/images/blog/Untitled-a91536dd-3ce4-49bf-bf0c-14ca9ca97447.png)

See it? No? Have a look at this image from MDN explaining the angle property of a rotate translation:

![](/assets/images/blog/Untitled-eb8ac4b6-88bb-42b8-a697-04e44738e7d2.png)

Yup, the marker's image was already rotated to 90 degrees. And I haven't noticed. Hey, now I can use my own magicAngleAdjusment method here and it be much simpler: `return angle - 90;`

This was an external influence. I have isolated the problem poorly and it cost me about 4 hours.

**Third step: Isolate the problem. Again. This time correctly. I hope.**

I created my own marker, with a little bit of a difference: it was a circle (fitted perfectly in a square container with a transparent background), with a line starting from its center to its top (from 50%, 50% to 50%, 0%):

![](/assets/images/blog/Untitled-cfb94cd0-b006-427c-979b-63d4113d1ff5.png)

With this I could make sure of two things: the marker is perfectly centered on its coordinate and it is rotated to match the correct angle. 

That didn't work. When rotated, the image's center was just a bit off:

![](/assets/images/blog/Untitled-04e3ef4e-dfc5-40f0-838c-cd480fd9eac7.png)
<span class="small text-center text-block">
_you're not suppose to see the red line under the marker_
</span>

**Fourth step: Check for problems in the logic that draws markers on the map**

3 hours out the window, spent with testing different scenarios.

**Time to repeat step #2.**

![](https://i.imgur.com/Bd121rw.gif)
<span class="small text-center text-block">
_OOOH, ok._
</span>

What's going on here? It looks like the center of the `<div>` I'm rotating shifts slightly when the div is rotated. The `<div>` doesn't have a width nor a height set, so it's expanding itself to contain the 30px X 30px image within. Combing through the CSS specifications, I couldn't find the reason on why this is happening, but my best guess is that it has something to do with transform-origin which defaults to 50% 50% (the center of the element). With no height and width, this is somehow not so accurately estimated if the element is rotated.

Adding a width and a height on the div fixes it.

**Fifth step, let's do this properly: expand the test to include more than 10 points, spread across the map**

After adding more points to the map, I had the unpleasant surprise to see that some of the markers where still not rotated correctly. Some where, some weren't, in what seemed to be a total random order. 

After repeating step #2 one last time, I noticed that some markers were off by about 5 degrees when pointing to other markers placed towards the south from them. Or to be super precise, if the latitude of the second point was smaller, the results were not accurate.

Without a shadow of a doubt, the formula I used to calculate the angle difference was not correct. Just when I felt really confident in my protractor using skills. 

**How that piece of code ended up in production this way?**

I was saying earlier that I got the formula from a project that was already in production, it was unit tested and integration tested and manually tested (the let's hop into the car and manually test this app kind). I was using it to see if a certain vehicle is no longer following a certain road -  if the angle between the direction of the vehicle and the bearing of the road as more than 30 degrees, then it's time to recalculate. 

**It's important to do a really small post-mortem here and think about why I missed it earlier:** I didn't spot it then because the elements where always right next to each other (the angle wouldn't have been calculated at all if the vehicle was farther than 50 meters from the road). And, oh well, I only tested it on roads between Bonn and Cologne (important to say that Cologne is north of Bonn). I did the mistake of *assuming* that my test cases are limited to the different types and topography of the roads.

Bottom line is that I simply didn't identify my test cases correctly. In fact I only had two tests: one against a point that was towards the North-West and one against a point in the North-East and nothing in the south.

The initial magicAngleAdjustment actually helped me realize this pretty fast, as it includes all the correct four cases. So, to be clear, these were not edge-cases, these were the normal cases. In this situation, an edge case would be two points with the same latitude or longitude or both.

![](/assets/images/blog/Untitled-40dd1fde-824b-41a7-bc9a-5144e21ad066.png)

So, as per the above image, I was only testing against points #1 and #2, instead of all four of them. Honest mistake! And a more honest learning opportunity. You can read all the books in the world, sometimes you just have to shoot yourself in the foot to learning something. I let myself carried away and let the maps and driving on the autobahn get in the way, instead of seeing that my test scenarios simply summed up to me having two pairs of number (the coordinates of the points) and their values relative to each other, like the IFs in the magic method.

Going map to the mathy mappy stuff*:* the formula I was using was for calculating the azimuth (an angle, measured clockwise between a line perpendicular to the north and a line passing through the first and the second point). Something like this:

![](/assets/images/blog/Untitled-fd0fae21-9978-433c-a9a7-6423a8a3ac02.png)

The problem with the azimuth formula I was using was that it doesn't take the curvature of the earth into consideration. Meaning that if you would have a distance of 10 kilometers between two geographical coordinates, and you would scatter some other points along it, the azimuth angle between each of the points and the end point would be different. It was calculating the angle on a straight line. The formula and more are beautifully explained here, in the Bearing section: 

[http://www.movable-type.co.uk/scripts/latlong.html](http://www.movable-type.co.uk/scripts/latlong.html)

Once I understood that, I did a few more connections and I was able to find the correct formula pretty quickly. About that: as I was saying, we flatten our maps using the MERCATOR projection so a straight line on our flattened map, upon re-sphering of the map, would become something that's called a [Rhumb line](https://en.wikipedia.org/wiki/Rhumb_line). One of the characteristics of a Rhumb line is that it represents a path with constant bearing relative to the North, so by calculating that bearing, we're sure to have the correct angle regardless of relative position or distance between the points. 

## All good things come to an end

Once the formula was in place, I was able to quickly add some unit tests, make sure the results were OK (I got to use that protractor once more) and send the results to the other developer. That was quite the ride and I learned some interesting stuff along the way, like how the azimuth [is used during wars](https://en.wikipedia.org/wiki/Indirect_fire) or [which one of the map projections is the best](https://www.youtube.com/watch?v=kIID5FDi2JQ) ([spoiler on that](https://i.imgur.com/O8poncb.jpg)). But I must say, the most fun part was using a protractor when writing unit tests.

Let's finally hit that park. Or not, 'cause it's raining. It always does around here, but at least, I'm not living on a boring, flat, Earth.
