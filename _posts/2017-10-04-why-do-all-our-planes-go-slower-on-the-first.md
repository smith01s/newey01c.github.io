---
layout: post
title: Why do our aircraft take off slowly on the first of the month?
comments: true
---

This was one of the more interesting discoveries of the last few weeks, and I
thought that it would make a fun post for the flight data community. As a part
of my role as a Data Scientist, it is one of my responsibilities to ensure that
the data we record and monitor is of high enough quality to base business
decisions on; that is, I have to make sure that our numbers make sense. As a
part of an investigation into data quality problems, I was looking into a
particular key point value[^1] called `Groundspeed with Gear on Ground Max` ---
which essentially is intended to record the maximum speed that an aircraft
reaches when taking off or landing.  Using this data, we can theoretically
detect situations which may damage the landing gear and reduce fuel efficiency
(and then alert the airlines accordingly).

<!-- more -->

To this end, I gathered a rather large dataset for this investigation (about 5
million flights from the last 18 months) and started to explore the
distribution of this particular KPV. One of the first ports of call is to look
at the average speed of aircraft over a given time period (in this case, *per
day*). This gives me an of how things are changing (or even *if* they're
changing --- they shouldn't!). To my surprise, I did find some periodic
fluctuations in this data; more specifically, the average takeoff speed across
*all aircraft* would drop by about 20% on two days every month, and this seemed
to be a regular occurrence. Let the head-scratching commence.


### A side note on data quality

Quality issues are very common with the sort of data that we deal with at
Flight Data Services; not only are the analysts and data scientists quite far
removed from the production of data (and thus certain trends can be difficult
to interpret without a lot of domain knowledge), but the raw data also passes
through several different transformations before any rigorous analysis is
performed (i.e. conversion from flight data readout format to engineering
units, signal processing and filtering, etc). This combination of factors (and
a rather complex analysis toolchain) means that there are several different
entry points for errors and it often requires a considerable amount of
sleuthing to discover the underlying cause of the problem --- this was no
exception.


## The problem

As you can see in the plot below, the average speed for flights drops
significantly on multiple occasions each month; quite frequently breaching the
99.5% prediction interval, which is obviously unexpected (and if real, quite
concerning). For those unfamiliar with the idea, a prediction interval is
simply a range that's calculated such that 99.5% of a dataset is expected to
lie within it; so, any data that falls *outside* of this range is quite
unusual.

![Averaged groundspeed with gear on ground](/images/groundspeed_interval.svg)

Drilling into this a little bit deeper, it does in fact appear that this
average speed drop phenomenon happens on the same days each month; on the 1st
and 11th of each month, to be precise. A clearer illustration of this is below
(left), with vertical lines to clearly show the dates. After a little bit more
investigation (and several very large cups of tea), I discovered that this drop
in average speed is caused by huge surge in the number of what I call
"zero-movement flights" - i.e. data in the flight recorder in which the
aircraft doesn't move. The plot thickens.

![Daily averages of groundspeed max KPVs](/images/groundspeed_max.svg)

This situation was a prime candidate for applying some of the investigative
techniques that I'd been employing in other projects, and I was able to apply a
machine learning technique known as a "decision tree" (the details of this are
quite technical, so I'll leave it as an exercise for the reader to investigate
further --- but [here][decision-trees-breiman] is a good starting point).
Essentially a decision tree is an algorithm that builds a tree-like diagram
that iteratively splits a dataset into smaller bits until it has highlighted
some of the attributes that are *most overrepresented* in a particular dataset
(by maximising a metric called *information gain*). Decision trees are a very
simple and very powerful machine learning technique; and they're ideally suited
for this sort of investigative modelling because it's super simple to visualise
them. Here's one I made for an earlier post;

![A very small sample decision tree](/images/decision_tree_diagram.svg)

[decision-trees-breiman]: https://www.stat.berkeley.edu/~breiman/randomforest2001.pdf

After building a couple of exploratory models and drawing up some
visualisations from the results, it turned out that one of our customers'
airlines was performing scheduled aircraft maintenance on the 1st and 11th of
each month. The aircraft would be switched on and tested or fixed (at which
point the flight data recorders would kick in, despite the aircraft not going
anywhere!). Further to that, the squat sensors in the landing gear would be
flicked on and off hundreds (or thousands) of times --- and due to the way that
we process flight data, these sensor events would register as individual
landings, even though the aircraft hadn't moved! The data management staff at
the airline are supposed to record these flights using a different kind of
flight record, but obviously someone hadn't got the memo! This had happened
every month regularly for approximately 18 months, meaning that there were
hundreds of thousands of these "zero-movement" flights just sitting there in
the database, skewing the average speeds that I was trying to investigate. Once
these flights were excluded from the dataset, the data looked much more
sensible (hooray!).

![Finally, a sensible distribution](/images/nice_histogram.svg)

Mystery solved.

[^1]: We call these "KPVs" for short --- essentially sensor values recorded at a
      specific point in a flight
