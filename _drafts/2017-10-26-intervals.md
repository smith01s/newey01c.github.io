---
layout: post
title: You're probably using the wrong kind of confidence interval
comments: true
---

Recently, I've taken to exploring some of the time-series data that we have on
file at Flight Data Services --- and was developing a fairly straightforward
anomaly detection algorithm. The data was flight statistics which had been
captured and aggregated at a daily frequency. My first approach to detecting
anomalies was to compute the mean and a confidence interval to see where the
data exceeds this interval --- data outside of this range would then be
classified as an anomaly. This is a pretty rudimentary statistical approach,
but one that's commonly applied in this sort of context and is generally
regarded to work well. The only problem is that this is actually *the wrong
thing to do™*.

<!-- more -->

## What does a confidence interval actually measure?

At the time, I wasn't really aware that there were multiple types of
statistical interval --- and most importantly. I didn't know *why* using
confidence intervals was the wrong tool for the job.

A confidence interval asserts that data is generated according to an underlying
normal distribution --- that is, that there is some noisy process that
generates data according to a probability density function that is assumed (in
this case) to be Gaussian. The problem is, the parameters of this distribution
are *unknown*, and as such, these parameters have to be estimated from samples.
This is easily done using maximum likelihood estimation, but because the
samples are not necessarily representative of the underlying distribution, this
measurement is uncertain.

<div>
$$
{\displaystyle \mathcal{N}(\mu, \sigma^{2})={\frac {1}{\sqrt {2\pi \sigma ^{2}}}} e^{-{\frac {(x-\mu )^{2}}{2\sigma ^{2}}}}}
$$
</div>
*<center>The definition of the univariate Gaussian distribution.</center>*
![Annotated Gaussian distribution (Wikipedia)](/images/normal_distribution.svg)

In short, confidence intervals are mis-applied here because a confidence
interval expresses the range of expected values *of the mean* of a set of
values. Did you get that? It's a subtlety, but an important one. A confidence
interval expresses the expected *range of mean values*. That is, a 95%
confidence interval suggests that if you sampled a population 100 times, then
the *sample mean* would lie within the confidence interval *95 out of those 100
times*. It's clear then, that basing an anomaly detection algorithm on this
kind of interval will lead to lots of false positives as the interval will lie
quite tightly around the mean.

### When to use it?

You should use a confidence interval if you're attempting to express your
confidence in a range of *expected values* for a particular random variable.
For example, if I wanted to estimate the average speed of cars driving past my
office during the home-time rush every day (*hint: it's not fast*), then I
would record a large number of measurements and take the average --- let's say,
10 kilometres per hour. A confidence interval (in this case, let's say a 95%
confidence interval of ±2km/h) is simply a measure of my expectation of
acceptable values of the mean speed --- that is, in 95% of cases I would expect
the *average* speed of a sample of cars to be between 8km/h and 12km/h.


## Alternatives to a confidence interval

After a little bit of reading up on other anomaly detection techniques, I
stumbled upon a couple of online resources that suggested using some other,
lesser-known intervals for estimating confidence of data. These are a little
bit more esoteric, but have a considerably firmer statistical grounding and
generate much more sensible intervals.

## Prediction interval

### What does it measure?

### How is it defined?

### When to use it?

## Tolerance interval

### What does it measure?

### How is it defined?

### When to use it?


[kmjn]: http://www.kmjn.org/notes/tolerance_intervals.html
