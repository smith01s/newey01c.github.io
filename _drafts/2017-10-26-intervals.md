---
layout: post
title: Confidence intervals considered (kinda) harmful
comments: true
---

Recently, I was tasked with doing some straightforward anomaly detection on a
large amount of time-series data, captured and aggregated at a daily frequency.
My first approach was to compute the mean and a confidence interval to see where
the data exceeds this interval --- and an exceedance would then be classified as
an anomaly. This was a really rudimentary statistical approach, but one that's
commonly applied in this sort of context and is generally regarded to work well.
The only problem is that this usage is actually *wrong*.

<!-- more -->

## What does a confidence interval measure?

At the time, I wasn't really aware of this application of confidence intervals
being wrong --- as far as I knew, it was a perfectly valid statistical
technique. In short, it's wrong because a confidence interval is a expresses the
range of expected values *of the mean* of a set of values --- not the expected
range of the overall data. The implication of a confidence interval, then, is
that the mean lies within a particular range --- and while the range of the rest
of the data is likely to be close to this (depending on variance) ---
statistically speaking, basing an anomaly detection algorithm on this will lead
to an invalid conclusion.

### When to use it?

You should *only* use a confidence interval if you're attempting to express
your confidence in a range of *expected values* for a particular random
variate. For example, if I wanted to estimate the average speed of cars driving
past my office during the home-time rush every day (hint: *it's not fast*),
then I would record a large number of measurements and take the average ---
let's say, 10 kilometres per hour. A confidence interval (in this case, let's
say a 95% confidence interval of ±2km/h) is simply a measure of my expectation
of acceptable values of the mean speed --- that is, in 95% of cases I would
expect the average speed to be between 8km/h and 12km/h.

### How is it defined?

This is where things get a little statistical, but it's important to know how
these intervals are derived --- because this will allow you to figure out which
is the most appropriate interval to use. A confidence interval asserts that
data is generated according to an underlying normal distribution --- that is,
that there is some noisy process that generates data according to a probability
density function that is (usually) assumed to be Gaussian. The problem is, the
parameters of this distribution are *unknown*, and as such, these parameters
have to be estimated from samples. This is easily done using maximum likelihood
estimation (that is, *estimating the mean and standard deviation from
samples*). However, because the samples are not necessarily representative of
the underlying distribution, this measurement is uncertain. In statistical
language, we only know the *sample mean*, and not the *population mean*.

<div>
$$
{\displaystyle \mathcal{N}(\mu, \sigma^{2})={\frac {1}{\sqrt {2\pi \sigma ^{2}}}} e^{-{\frac {(x-\mu )^{2}}{2\sigma ^{2}}}}}
$$
</div>
*<center>The definition of the univariate Gaussian distribution.</center>*

The confidence interval then, essentially the interval between which we expect
the *population mean* to lie --- and this can be estimated using the *sample
mean*. We estimate the parameters of particular probability distribution by
calculating the mean and standard deviation of a given sample (i.e. some
measurements), and then we use those parameter estimates to make inferences
about the population overall. What we need to do, then, is take into account
the inherent uncertainty in the sample.

![Annotated Gaussian distribution (Wikipedia)](/images/normal_distribution.svg)


## Alternatives to a confidence interval

After a little bit of reading up on other anomaly detection techniques, I
stumbled upon a couple of online resources that suggested using some other,
lesser-known intervals for estimating confidence of data.

## Prediction interval

### What does it measure?

### How is it defined?

### When to use it?

## Tolerance interval

### What does it measure?

### How is it defined?

### When to use it?


[kmjn]: http://www.kmjn.org/notes/tolerance_intervals.html
