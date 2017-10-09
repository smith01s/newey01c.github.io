---
layout: post
title: Why do aircraft take off more slowly on the first day of the month?
comments: true
---

This was one of the more interesting discoveries of the last few weeks, and I
figured that it would make a fun entry into the scrapbook. As a part of an
investigation into data quality problems, I was looking into a particular key
point value[^1] called `Groundspeed with Gear on Ground Max` --- which
essentially is intended to record the maximum speed that an aircraft reaches
when taking off or landing. I gathered a rather large dataset for this purpose;
containing about 5 million flights from the last 18 months, and started to
explore the distribution of this particular KPV. Much to my annoyance, I found
some strange periodic fluctuations in this data. More specifically, the average
takeoff speed of aircraft would drop by about 20% on the *1st and 11th of each
month*. How frustrating.

<!-- more -->

## Data Quality

Quality issues are very common with the sort of data that we deal with at
Flight Data Services - not only are the analysts and data scientists quite far
removed from the production of data, it also passes through several different
transformations before any rigorous analysis is performed (i.e. conversion from
flight data readout format to engineering units, signal processing and
filtering, etc). This combination of factors means that there are several
different entry points for errors, and it often requires a considerable amount
of sleuthing to discover the underlying cause - this case was no exception.


## The Problem

As you can see in the plot below, the average speed for flights drops
significantly twice per month - often actually breaching the 95% prediction
interval (which is obviously uncommon - and perhaps a cause for concern).

![Averaged groundspeed with gear on ground.](/images/groundspeed_interval.svg)

Drilling into this a little bit deeper, it does in fact appear that this
happens regularly - on the 1st and 11th of each month, to be precise. A clearer
illustration of this is below (left), with vertical lines to clearly show the
dates that this phenomenon occurs on. After a little bit more investigation, it
appears that this drop in average speed is caused by huge surge in the number
of what I call "zero-movement flights" - i.e. data in the flight recorder in
which the aircraft doesn't move. The plot thickens.

![Daily averages of groundspeed max KPVs](/images/groundspeed_max.svg)

This was a prime candidate for applying some of the investigative techniques
that I'd been employing in other projects, and (after a liberal application of
decision trees) it turned out that one of our airlines (whose data we collect
and monitor) was performing scheduled maintenance on their aircraft on these
dates each month. The aircraft would be switched on and tested or fixed (at
which point the flight data recorders would kick in, despite the aircraft not
going anywhere!). This had happened every month regularly for approximately 18
months - meaning that there were tens of thousands of these "zero-movement"
flights just sitting there in the database, skewing the average speeds that I
was trying to investigate. Once I had figured out a way to exclude these, the
data looked much more sensible.

Mystery solved.

[^1]: We call these "KPVs" for short - essentially sensor values recorded at a
      specific point in a flight)
