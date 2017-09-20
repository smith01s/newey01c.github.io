---
title: Quality Issues with Flight Data
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

Typically, errors are introduced in the conversion from the data frame (i.e.
data from the flight data recorder) to engineering units. There are lots of
different ways that errors can be introduced, but essentially data is encoded
when going into the flight data recorder - often in non-standard ways that
aren't always documented correctly by the aircraft manufacturer - or that get
misinterpreted by the data frame developers (i.e. the people who write the
frame layouts for decoding data). As such, a sizeable proportion of errors
(e.g. false positives for events and so forth) come from this data decoding
process. This happens very frequently, as most of the "interesting" data
revolves around extrema - that is, analysts are mostly interested in
particularly high or low sensor values as they usually indicate something that
went wrong on a particular flight.

