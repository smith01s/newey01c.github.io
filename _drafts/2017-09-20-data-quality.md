---
title: On Quality Issues with Flight Data
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
contain thousands of sensors - often recording *thousands of parameters* for
each flight. In terms of sheer pr


### Solution: Imputation and Sensor Fusion
