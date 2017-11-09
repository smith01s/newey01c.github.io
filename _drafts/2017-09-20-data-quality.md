---
title: On Data Quality in Flight Data Analytics
layout: post
comments: true
---

One of the core difficulties with analysing flight data is that analytics is
very far removed from the initial production and ETL of data. Before any flight
data reaches a data scientist, it has to pass through various teams of
specialists to be converted into sensible formats for archival. This often
leads to a case of "too many cooks" - where too many processing stages make
any analytics very fragile, because most of the "interesting" data simply can't
be relied upon to be accurate.

<!-- more -->

## The Flight Data ETL Pipeline - A Summary

* Flight data is recorded from aircraft in-situ, using hardware that interfaces
  with the Flight Data Recorder (FDR).
* After each flight, data is downloaded from the secondary FDR, and sent via
  the internet (or sometimes on physical media) to a centralised data store.
* The aircraft operator then sends Achieved Flight Records (AFRs) to the flight
  data company. These AFRs contain valuable metadata about the flight (crew
  information, flight schedules, etc).
* At this stage, sensor data is still in its (heavily compressed and
  proprietary) binary data format. The FDR data downloaded earlier is passed
  onto a team of specialists that use a data schematic called a Logical Frame
  Layout (LFL) to convert the raw binary data into a sensible, interchangeable
  data format (usually something like HDF5). This is difficult because most
  aircraft model use a slightly different data specification, and the
  specifications are given to the specialists by the aircraft manufacturer.
  Unfortunately, these specifications aren't always *correctly written by the
  manufacturer*.

## Sources of Error

### Conversion Errors

Typically, errors are introduced in the conversion from the data frame (i.e.
data from the flight data recorder) to engineering units - but this is by no
means the *only* source of error. There is a myriad of different ways that
errors can be introduced, but a key point is when sensor data is encoded and
sent to the flight data recorder. This encoding process is often done in
non-standard ways, and these aren't always correctly documented by the aircraft
manufacturer (or correctly interpreted by the developers that write the
decoding software). As such, a sizeable proportion of errors (e.g. false
positives for events and so forth) are essentially artefacts from the lossy
encoding/decoding of sensor data.

This effect is exacerbated because most "interesting" data revolves around
looking into extreme values. That is, event detection systems are mostly
interested in particularly high or low values - as these usually indicate
something abnormal on a particular flight. Unfortunately, these data encoding
problems often generate signal artefacts that look like events (to an automated
system, at least) - making our job much harder. For example, a momentary break
in a signal for an airspeed sensor would look like a complete loss of speed to
an automated system - and this is why we need analysts!

#### Solution: Descriptive Statistics, Exploratory Analytics


### Data Acquisition Unit (DAQ) Errors

Another common source of problems is from hardware issues relating to the
physical limits on some of the recording devices. Some older systems, for
example, record flight data onto optical media (i.e. CDs or DVDs) - optical
media are great for long-term data storage and archival, but unfortunately
aren't great for robust data recording. As well as being susceptible to
tracking problems when encountering corrupted sectors on disks, occasionally
flights that experience large amounts of turbulence will actually cause the
rotating optical platter to jog and skip - meaning that the recording unit will
jump over some sectors on the disk. This can cause various problems, including
duplicated data, corrupt sectors, and substantial gaps in recordings.

#### Solution: Break Detection


### Sensor Errors

Another place that errors can be introduced is simply through faulty sensors.
Certain varieties of sensor can cause more problems than others - for example,
a faulty airspeed sensor (an analogue sensor that records speed values in
knots) might yield nonsense or occasionally exhibit short data spikes. These
spikes may trigger *some* spurious events, but this doesn't usually pose a huge
difficulty.

Another (and often more problematic) fault is in binary mechanisms like the
"gear on ground" squat sensors embedded in the landing gear of various
aircraft. A faulty switch in this context can potentially report changes in the
state of the landing gear thousands of times in a flight (i.e. it can generate
*thousands* of gear up/down events!). This spurious information can cascade
through the analysis systems and cause problems with some of the other derived
parameters (for example, ground speeds can register at 600 knots if the
aircraft thinks that the landing gear is down during cruise!).

As aircraft are extremely complex feats of engineering, a single aircraft can
contain tens of thousands of sensors - and often record *several thousand
parameters* for each flight. This is likely to cause problems, as --- simply
due to the law of large numbers --- even if 99.9% of sensors are being recorded
correctly --- generally *something* will still be reporting incorrect values.
It is part of our job as data scientists to figure out where this invalid data
is hiding, and take action to ensure that none of the analysts or airlines draw
false conclusions from garbage data.


### Solution: Masking, Imputation and Sensor Fusion

This action can take various different forms; the simplest being just marking a
particular stream of data as invalid. This is particularly helpful if there are
recorded time-series parameters with gaps in signal or random data spikes, but
in other cases (e.g. excessive random noise), there are other approaches we
might want to take so that we don't trigger events unnecessarily.

In most cases, fixing or imputing noisy data simply requires an application of
some smoothing techniques (moving averages and so on). However, this doesn't
necessarily work in where the data are missing altogether --- in this sort of
situation, there are a few viable options; one is masking the missing data out
(i.e. adding a placeholder "fill value" that makes it explicitly clear that the
data are missing), and another is imputation based on machine learning or
statistics. This can often be done with Kalman filters and the expectation
maximisation (EM) algorithm, or with some more specialised techniques like
autoregression/ARMA models.

Another option (and arguably, the best of all) is to use knowledge of
correlated parameters to infer what the missing data *could* look like. For
example, a good indicator of an increase in nose pitch might be normal G (i.e.
"upwards" acceleration on the aircraft) --- and likewise, a good indicator of
airspeed might be engine N1 and altitude. Combining a number of correlated
parameters into an estimator for missing data can theoretically produce a very
powerful model for imputing data.

However, the ability to impute realistic and sensible-looking data raises a
number of questions. While we *might* be able to take an educated guess about
what some missing data should look like, this is unlikely to be an exact
analogue of the conditions that the aircraft is operating in. We might be able
to take a good guess at a missing airspeed value for 10 seconds (or perhaps
even 10 minutes), but there's a reasonable chance that our imputation will be
wrong --- particularly if the missing data occurred while the aircraft was
operating outside of normal conditions. It must be made very clear then, that
conclusions about air safety policy, standard operating procedures, etc,
shouldn't be made based on artificially-created data that *simply didn't exist
in the first place*.


# Summary

Whew, that was wordy! I hope you managed to stay awake all the way to the end
--- and I hope that you now have a reasonable understanding of some of the
quality problems that we experience in flight data analysis. If I've learned
one thing so far since working at Flight Data Services: it's *never* simple.
