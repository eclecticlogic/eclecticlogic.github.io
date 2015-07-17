---
layout: post
title: Curve Fitting
author: Karthik Abram
tags: algorithm
---

Here is a simple technique to modify linear data points to a curve of your choice. If your function looks linear like this:

![]({{ site.baseurl }}public/images/blog/curve/linear-1.png)

or even has a inverse linear form like this:

![]({{ site.baseurl }}public/images/blog/curve/linear-2.png)

And you want to instead map it to a curve that accentuates the initial values and suppresses the tail:

![]({{ site.baseurl }}public/images/blog/curve/curve-1.png)

or has an exponential drop at the end points like this:

![]({{ site.baseurl }}public/images/blog/curve/curve-2.png)

then, Cubic Bezier curves can be used to achieve the desired transformation. A Cubic Bezier curve is given by:

B(t) = (1 - t)<sup>3</sup>**P<sub>0</sub>** + 3(1 -t)<sup>2</sup>t**P<sub>1</sub>** + 3(1 -t)t<sup>2</sup>**P<sub>2</sub>** + t<sup>3</sup>**P<sub>3</sub>**

Here P<sub>0</sub>, P<sub>3</sub> are the end-points of the curve and P<sub>1</sub> and P<sub>2</sub> are control points. A Cubic Bezier curve, at the starting point is always tangential to the line segment P<sub>0</sub>, P<sub>1</sub> and in the direction of P<sub>0</sub>, P<sub>1</sub>. At the endpoint, the curve is tangential to P<sub>2</sub>, P<sub>3</sub> and is in the direction of P<sub>2</sub>, P<sub>3</sub>. You can play around with various Cubic Beziers shapes at [desmos.com](https://www.desmos.com/calculator/cahqdxeshd).

When implementing a Cubic Bezier curve, the first oddity you will encounter is the fact that unlike polynomial equations, the Bezier does not give you y-coordinates as a function of x-coordinates. Instead, it independently gives you x and y coordinates as a function of a parameter t which varies from 0 to 1. When t = 0, the curve is at point P<sub>0</sub> and when t = 1, the curve is at point P<sub>3</sub>. Worse, when you vary t linearly, you do not get linear increments of x. Instead, you get linear increments along the curve's length. In fact, it is this property of Bezier curves that makes them suitable as easing functions for animations. By constructive a Cubic Bezier with steep starting and ending slopes, you can get a slow start in the x axis followed by an acceleration in the middle and a gradual slowdown towards the end. But for mapping a traditional function of the form y = f(x), we would like to transform the Bezier to a similar function.

There is no good mathematical way to achieve this given expression like t<sup>2</sup> and t<sup>3</sup> in the equation. Instead, one can approximate a solution by doing a binary search along t, computing the corresponding x until we are within a tolerance limit. At that point, the value of y can be computed. The gist below shows a Java program to do that. In addition, the program allows you to pre-compute y = Bezier(x) for x = (start, end) in specified increments.

{% gist eclecticlogic/e46120cb719d38f5360e CubicBezierCurve.java %}


The class above can be extended to create a composite Cubic Bezier curve fitting algorithm that combines multiple Cubic Bezier curves to model any shape you want to create. 
