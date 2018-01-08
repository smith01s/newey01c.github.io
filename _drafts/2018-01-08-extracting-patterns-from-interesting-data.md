---
layout: post
title: The link between invalid events and shopping baskets
comments: true
---

One of our main jobs at FDS (as a flight safety company) is to generate events
and alerts when the aircraft that we monitor exceed certain tolerances. For
example, a customer might want us to monitor if their pilots regularly fly
outside of their allocated altitude range (i.e. a *level bust*) --- which is
largely a safety issue, as this is a risk factor for mid-air collisions.
Similarly, an airline might want to know if an engine runs too hot for too long
(as this will increase maintenance costs and *potentially* cause safety
issues).

<!-- more -->

So, what's the point of this post? Well, as a company, we're interested in
generating as many events as possible --- that is, we set our event thresholds
with a *very low tolerance* for deviation or error. The point of doing this is
to flag up as many potential safety problems as possible, and thus to hopefully
catch all genuine safety events.

The problem is that setting these thresholds at such low values causes a
considerable number of false positives (i.e. invalid events) --- and for each
event that gets generated, a trained human (i.e. an analyst) has to manually
inspect the data and make sure that the event was genuine. This often ends up
not being the case, as there are various causes of unusual patterns of data ---
and thankfully, very few of them are actually due to pilot error or
malfunctioning aircraft.

In actual fact, the events that we see are more likely to be *invalid* ---
potentially caused by faulty sensors, random interference in the electrical
systems in the aircraft, the submission of an incorrect flight record, or any
one of a hundred other things[^1]. This makes false positives one of the
largest cost centres of the company, as they're expensive to validate --- and
this also reduces the time available to each analyst for validating *real*
events.

[^1]: Actually, one of the more interesting phenomena is that of *operational
      invalidity* --- this is a situation where, for example, the location of
      an airport (and the local geography) may necessitate certain unusual
      flying patterns. A pretty good case study for this is Bardufoss airport
      in a particularly mountainous part of Norway --- the safest way to
      approach is to descend extremely quickly (which tends to trigger a host
      of automated safety events).


## Identifying the causes of invalid events

### Beer and nappies

This ([probably apocryphal][bn]) story

[bn]: https://www.theregister.co.uk/2006/08/15/beer_diapers
