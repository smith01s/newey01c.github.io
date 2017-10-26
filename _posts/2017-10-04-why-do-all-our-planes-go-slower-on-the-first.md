---
layout: post
title: Why do aircraft take off more slowly on the first day of the month?
comments: true
---

This was one of the more interesting discoveries of the last few weeks, and I
thought that it would make a fun post for the flight data community. As a part
of my role as a data scientist, it is one of my responsibilities to ensure that
the data we record and monitor is of high enough quality to base business
decisions on --- that is, I have to make sure that our numbers make sense. As a
part of an investigation into data quality problems, I was looking into a
particular key point value[^1] called `Groundspeed with Gear on Ground Max` ---
which, essentially, is intended to record the maximum speed that an aircraft
reaches when taking off or landing.  Using this data, we can theoretically
detect situations which may damage the landing gear and reduce fuel efficiency
(and then alert the airlines accordingly).

<!-- more -->

To this end, I gathered a rather large dataset for this investigation --- about
5 million flights from the last 18 months --- and started to explore the
distribution of this particular KPV. The first port of call is to look at the
average speed of all aircraft *per day* --- this gives me an idea of how much
things change over time, if at all (they shouldn't!). Much to my annoyance, I
found some periodic fluctuations in this data. More specifically, the average
takeoff speed across *all aircraft* would drop by about 20% on the 1st and 11th
of each month. Let the head-scratching commence.

### A side note on data quality

Quality issues are very common with the sort of data that we deal with at
Flight Data Services --- not only are the analysts and data scientists quite far
removed from the production of data (and thus certain trends can be difficult
to interpret without a lot of domain knowledge), but the raw data also passes
through several different transformations before any rigorous analysis is
performed (i.e. conversion from flight data readout format to engineering
units, signal processing and filtering, etc). This combination of factors (and
a rather complex analysis toolchain) means that there are several different
entry points for errors, and it often requires a considerable amount of
sleuthing to discover the underlying cause of the problem --- this was no
exception.


## The problem

As you can see in the plot below, the average speed for flights drops
significantly on multiple occasions each month --- actually quite frequently
breaching the 99.5% prediction interval (which is obviously uncommon --- and if
real, quite concerning).

![Averaged groundspeed with gear on ground](/images/groundspeed_interval.svg)

Drilling into this a little bit deeper, it does in fact appear that this
average speed drop phenomenon happens quite regularly --- on the 1st and 11th of
each month, to be precise. A clearer illustration of this is below (left), with
vertical lines to clearly show the dates. After a little bit more investigation
(and several large cups of tea), I discovered that this drop in average speed
is caused by huge surge in the number of what I call "zero-movement flights" -
i.e. data in the flight recorder in which the aircraft doesn't move. The plot
thickens.

![Daily averages of groundspeed max KPVs](/images/groundspeed_max.svg)

This situation was a prime candidate for applying some of the investigative
techniques that I'd been employing in other projects, and I was able to apply a
machine learning technique known as a "decision tree" (the details of this are
quite technical, so I'll leave it as an exercise for the reader to investigate
further --- but [here][decision-trees-breiman] is a good starting point).

[decision-trees-breiman]: https://www.stat.berkeley.edu/~breiman/randomforest2001.pdf

After building a couple of exploratory models and drawing up some basic
visualisations, it turned out that one of our customers' airlines was performing
scheduled maintenance on their aircraft on these dates each month.  The aircraft
would be switched on and tested or fixed (at which point the flight data
recorders would kick in, despite the aircraft not going anywhere!).  The data
management staff at the airline are supposed to record these flights slightly
differently by using a different kind of flight record --- but obviously someone
hadn't got the memo! This had happened every month regularly for approximately
18 months --- meaning that there were tens of thousands of these "zero-movement"
flights just sitting there in the database, skewing the average speeds that I
was trying to investigate. Once I had figured out a way to exclude these, the
data looked much more sensible.

![Finally, a sensible distribution](/images/nice_histogram.svg)

Mystery solved.

[^1]: We call these "KPVs" for short --- essentially sensor values recorded at a
      specific point in a flight)
