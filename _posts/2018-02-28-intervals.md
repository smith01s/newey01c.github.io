---
layout: post
title: A crisis of confidence (intervals)
comments: true
---

Lately I've taken to exploring some of the aggregated event statistics that we
have on file at Flight Data Services --- and as a side project, I was using my
knowledge of statistics to develop a fairly straightforward anomaly detection
algorithm. My first approach was to compute the daily average of a particular
key point value (in this case, `Acceleration Normal at Touchdown`), and then
compute the mean and an arbitrary confidence interval (say, 99.5%). Data that
fell outside of this interval would then be marked as an anomaly and would
warrant further investigation. This is a pretty rudimentary statistical
approach to anomaly detection, but one that's commonly applied in this context
--- the only problem is that *it's wrong*.

<!-- more -->

## What does a confidence interval actually measure?

In the case, I'm going to be talking about the Gaussian distribution, but this
applies to other distributions too. In basic inferential statistics, we try to
use data generated from a *sample* of a population to make conclusions about
the population as a whole. That is, we are implicitly making the assumption
that the population's data is generated due to some underlying probability
distribution. The problem is, the parameters of this distribution (i.e. in the
Gaussian case, the mean and standard deviation) are *unknown* --- we can only
estimate those parameters by calculating statistics from *samples* of the
population.

These parameters (i.e. mean and std. dev) are easy to calculate from *sample
data*, but the samples are not necessarily representative of the underlying
population --- so these estimates have an inherent amount of uncertainty in
them. One other thing that's useful to note is that because of a neat
statistical property known as the *law of large numbers*, we can be *more*
confident that a large sample better represents the underlying population than
a small one. So bigger is better --- at least in statistics.

![Annotated Gaussian distribution (Wikipedia)](/images/normal_distribution.svg)

A confidence interval, then, is a mathematical concept that expresses the
*uncertainty* in a particular estimate of the population's distribution.
Phrased another way, the upper 95% confidence bound is the point at which
*97.5%* of the "weight"[^1] of the probability density function lies *below* it
(and of course, the opposite is true for the lower bound). If you haven't seen
this before, you might be thinking "why 97.5%"? Well that's because a
confidence interval is *two-sided* --- so $ 100\% - 95\% = 5\% $, and
$\frac{5\%}{2} = 2.5\%$. Of course, then $100\% - 2.5\% = 97.5\%$.


## What was I doing wrong?

In short, I was applying confidence intervals incorrectly because a confidence
interval expresses the range of expected values *of a sample mean*. It's a
subtle detail (and one that is often missed, myself included!), but an
important one. A 95% confidence interval suggests that if you sampled a
population 100 times, then the *sample mean* would lie within the confidence
interval *95 out of those 100 times* --- and, from a Bayesian perspective, you
can be 95% sure that the *population mean* lies within the 95% confidence
interval. Because a confidence interval is about the expected range of values
*of an average*, basing an anomaly detection algorithm on this kind of interval
will lead to lots of false positives. If you think of each new flight as a new
sample of *one* (as opposed to an average of *several*), you should hopefully
be able to see why a confidence interval is too small --- because the variance
of a sample containing only a single data point will be very high!

<div>
$$
{\displaystyle \mathcal{N}(\mu, \sigma^{2})={\frac {1}{\sqrt {2\pi \sigma ^{2}}}} e^{-{\frac {(x-\mu )^{2}}{2\sigma ^{2}}}}}
$$
</div>
*<center>The definition of the univariate Gaussian distribution.</center>*


### When to use it?

You should use a confidence interval if you're attempting to express your
confidence in a possible range for the *expected values* of a particular random
variable. For example, if I wanted to estimate the average speed of cars
driving past the office during the home-time rush every day (hint: *it's not
fast*), then I would record a large number of measurements and take the average
--- let's say, 10 kilometres per hour. A confidence interval (in this case,
let's say a 95% confidence interval of ±2km/h) is simply a measure of my
expectation of the range of values for the average speed --- that is, 95% of
the time, I would expect the *average* speed of a sample of similar cars to be
between 8km/h and 12km/h.


## What about the *tolerance interval*?

After a little bit of reading up on other anomaly detection techniques, I
stumbled upon a couple of online resources that discussed some lesser-known
intervals that have a slightly stronger statistical justification for building
anomaly detection thresholds --- and so produce much more sensible results. In
this case I'm going to be talking about the *tolerance interval* --- an
interval that works as a sort of "set-and-forget" quantity that is estimated
once and then can be re-used for future observations (this differs from
something like the *prediction interval* which should technically be
re-computed each time).

Instead of estimating the range of possible values that contains the mean of a
population, tolerance intervals attempt to estimate the range of values that
the *entire* population resides in. This is a conceptual departure from the
usual confidence interval, because one must also provide a *tolerance*
threshold as well as a *confidence level*. That is, a tolerance interval
answers the question "what range will contain 99% of the population, with a 95%
confidence?". That essentially means that, 95% of the time, 99% of data points
will lie within the tolerance interval.


### How is it defined?

This is where things start to get fun. As mentioned above, we need to provide
*two* parameters to calculate the tolerance interval --- and that's because we
now need to consider *two* probability distributions; the Gaussian (as before),
and the Chi-squared distribution as well. In short, the Chi-squared
distribution (with $n$ degrees of freedom, where $n$ is the number of data
points in a sample) shows the probability distribution of the sum of squares of
individual data points. The tolerance interval $t_i$ can be approximated as
follows, where $n$ is the degrees of freedom, $z \frac{1 - p}{2}$ is the
required *confidence level*, and $\gamma$ is the proportion of data that will
lie within the interval $t_i$;

$$ t_i = \sqrt{\frac{n \left(1 + \frac{1}{N}\right)
z^2_{\frac{1-p}{2}}}{\chi^2_{1-\gamma, \, n}}} $$


## An example

In Python, an approximation to the tolerance interval can be computed using
just a few lines of code;

```python
import numpy as np
import scipy.stats as st

def tolerance(prop, conf, n):
    v = n - 1
    z = st.norm.ppf((1 - prop) / 2)
    chi = st.chi2.ppf(1 - conf, v)
    return np.sqrt((v * (1 + 1 / n) * (z ** 2)) / chi)
```

And of course, applying this tolerance interval to 1,200 randomly-generated
numbers yields the following bounds (note how few outliers there are!).

![Some data with tolerance intervals applied.](/images/tolerance_interval.svg)

This figure was generated quite easily using the following Python code;

```python
import matplotlib.pyplot as plt

# generate synthetic data
n = 1200
X = np.linspace(1, n, n)
Y = np.random.randn(n)

# compute tolerance interval
ti = tolerance(0.99, 0.95, n)
# de-normalise interval using standard deviation
d = Y.std() * ti

# compute actual bounds on real line
lb = Y.mean() - d
ub = Y.mean() + d

# plot!
plt.scatter(X, Y, alpha=0.5)
plt.axhline(lb)
plt.axhline(ub)
```

## In summary

Tolerance intervals are a very powerful statistical tool for creating
"set-and-forget" thresholds for performing rudimentary anomaly detection on
certain types of statistical data. There are some great resources on these
statistical concepts[^2][^3][^4], so if my explanation falls a little bit short
(and it almost certainly does), I'd check those out.


# References and Footnotes

[nist]: http://www.itl.nist.gov/div898/handbook/prc/section2/prc263.htm
[kmjn]: http://www.kmjn.org/notes/tolerance_intervals.html
[amst]: https://amstat.tandfonline.com/doi/abs/10.1080/00031305.1992.10475882

[^1]: I know that's the wrong word (sorry, mathematicians)
[^2]: [Tolerance intervals for a normal distribution][nist]
[^3]: [You might want a tolerance interval --- KMJN.org][kmjn]
[^4]: [What About The Other Intervals?][amst]
