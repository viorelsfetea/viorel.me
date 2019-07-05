---
layout: post
title: 'Problem solving exercise: find the middle/centroid of a polyline'
summary: 'Recently, I got a task to add some direction arrows on a set of polylines that were drawn on a map. After quickly testing some code, I found that the most esthetically pleasant option would be to place this arrow in the middle of the polyline. '
date: 2019-07-05 12:00:18
comments: false
category: techy
author: Viorel
slug: find-the-middle-centroid-of-a-polyline
---

[Jump directly to the code (*if you must*)](#the-code). It's in Javascript, BTW.

Recently, I got a task to add some direction arrows on a set of polylines that were drawn on a map. After quickly testing some code, I found that the most esthetically pleasant option would be to place this arrow in the middle of the polyline. 

The problem initially seemed a little bit overwhelming because of what a polyline actually is: a list of coordinates at varying distances from one another. So, get a middle point between two points? No problem. Get a middle point between 2...n points. Mmmm, problem. But then I remembered what I tell my son at least once a day: when you have a big, overwhelming problem, it always helps to sit down and try to split it into smaller, more manageable problems. Oh yeah, and I also tell him to practice what he preaches. It's funny I forgot that, since effective problem solving is, well, the bread and butter of my job.

Anyhow, by practicing what I preach, I sat down and I "problem solved" the problem:

**Problem:** find the middle point of a set of geographical coordinates 

Let's simplify the polyline before starting: *A ——— B —————————— C —— D*

1. First step: get the total length of the polyline, i.e. the sum of all distances between each point in the polyline and the next point in the list: AB+BC+CD
2. Second step: get half the length of the polyline (divide the length in the first step to two): AB+BC+CD/2: *A ——— B ————X————— C —— D*
3. Third *and most important step*: find the points on the polyline that are *right before* and *right after* the point we want to find. We know the exact coordinates of these points and together they draw a line. The middle point we need to find is on this line. The problem is now simplified from "Find the middle of a polyline" to "Find a point on a line knowing the distance to the start/end of the line". When you get the points, you should also get the distances between them (really useful in the next step)
4. Finally: [draw the rest of the ow...](https://i.imgur.com/9Ywu3Bz.jpg) I mean, knowing the definition of the point, apply some maths to find its exact coordinates. I think the easiest way to go is some magic [using scalar multiplication on the vector between the two known points.](https://math.stackexchange.com/a/2045181)

**And the four pieces of code now:**

Important note about calculating the length: what formula you use to calculate this length depends on your project needs. In my particular case, I had a lot of short-distance polylines to shown on the screen all at once, so I opted to use the Euclidean distance. This has the disadvantage of yielding inaccurate results for longer distances due to the curvature of the earth. If that's your case, you need to use something that calculates the great-circle distance between two points. For that, you would need to use the Haversine formula.

So, the options:

1. Euclidean distance: good for short distances. The fastest option, if you need that, but inaccurate on longer distances. *If you are a flat-earther, this is really your only option.*
2. Haversine distance: good for any case, a little slower, not by much, you might need it, but probably not.

**I'm just going to assume you're using some kind of coordinate system with X and Y, where X is the longitude and Y is the latitude. *Really really important: X=longitude / Y=latitude***

*Step 1: calculate the total length of the polyline*
<a name="the-code"></a>
{{< highlight js >}}
    function getEuclideanDistance(pointA, pointB) {
        const distanceX = pointA.x - pointB.x;
        const distanceY = pointA.y - pointB.y;
      
        return Math.sqrt(distanceX * distanceX + distanceY * distanceY);
    };
      
    function getPolylineLength(points) {
        let distance = 0;
    
        points.forEach((point, index) => {
        if (typeof points[index + 1] === 'undefined') return;
        distance += getEuclideanDistance(point, points[index + 1]);
        });
    
        return distance;
    }
{{< /highlight >}}

*Step 2: get half of the length from step 1 (like, just divide it by 2). I'll include this in the next chunk of code*

*Step 3: get the points before and after. Iterate through all the points*

{{< highlight js >}}
    function getPointsCharacteristics(points) {
        const distanceToMiddlePoint = getPolylineLength(points) / 2;
    
        if (distanceToMiddlePoint === 0) {
          return points[0];
        }
    
        if (distanceToMiddlePoint < 0 || points.length < 2) {
          return null;
        }
    
        let distanceToNextPoint = 0;
        let distanceToPreviousPoint = 0;
        let pointIndex = 0;
    
        for (let i = 1; i <= points.length; i += 1) {
          distanceToPreviousPoint = distanceToNextPoint;
    
          distanceToNextPoint += getEuclideanDistance(points[i], points[i - 1]);
    
          if (distanceToNextPoint >= distanceToMiddlePoint) {
            pointIndex = i;
            break;
          }
        }
    
        if (distanceToNextPoint < distanceToMiddlePoint) {
          return null;
        }
    
        return {
            previousPoint: points[pointIndex - 1],
            nextPoint: points[pointIndex],
            distanceToPreviousPoint,
            distanceToNextPoint,
            distanceToMiddlePoint
        };
    }
{{< /highlight >}}

*Step 4: do the maths*

{{< highlight js >}}
    function getPolylineMiddlePointCoordinates(points) {
        const {
            previousPoint,
            nextPoint,
            distanceToPreviousPoint,
            distanceToNextPoint,
            distanceToMiddlePoint
        } = getPointsCharacteristics(points);
    
        const vectorScalingFactor = (distanceToMiddlePoint - distanceToPreviousPoint) / (distanceToNextPoint - distanceToPreviousPoint);
        
        return {
            x: previousPoint.x + (nextPoint.x - previousPoint.x) * vectorScalingFactor,
            y: previousPoint.y + (nextPoint.y - previousPoint.y) * vectorScalingFactor
        };
    }
{{< /highlight >}}

*You can test it with:*

{{< highlight js >}}
    getPolylineMiddlePointCoordinates([{x: 7.146979, y: 50.67989}, {x: 7.14688, y: 50.679924}, {x: 7.1467, y: 50.6799}]);
    //should return {x: 7.146841878732044, y: 50.67991891716427}
{{< /highlight >}}
