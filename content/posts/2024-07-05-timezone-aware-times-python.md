+++
title = "Timezone Aware Timestamps in Python"
date = 2024-07-06T12:14:00+02:00
tags = ["datetime", "timezone"]
categories = ["Python"]
draft = false
author = "Morten Kjeldgaard"
description = "Examples of how to use timezone aware datetimes in Python"
keywords = "python datetime pytz"
+++

The datetime object in Python is very powerful, but if you want to do serious timeseries analysis and use data from around the world, you need to use timezone aware `datetime` objects, and these are not the default.

```python
>>> import datetime as dt
>>> now = dt.datetime.now()
>>> print(now)
2024-07-05 16:03:40.991009
```

This is the current time, however the datetime object we have is not timezone aware, and the `tzinfo` object has the value `None`:

```python
>>> print(now.tzinfo)
None
```


## The trusty old way: The `pytz` module {#the-trusty-old-way-the-pytz-module}

Let us assume the dates you have are from a dataset and you determine they are relative to the `US/Eastern` timezone. We need to add the timezone information to the datetime object, and this can be done most easily using the `pytz` module. The `datetime` module can also deal with timezones, but it is cumbersome and convoluted to use by itself. With the `pytz` module it becomes much easier, and this module is very popular. However, `pytz` is not a module in Python's standard library, so you need to install it using `pip install pytz` or perhaps the package manager of your OS.

```python
>>> import datetime as dt
>>> now = dt.datetime(2024, 7, 5, 16)
>>> print(now)
2024-07-05 16:00:00
```

The time is given by date and hours and minutes, but there is no information about the timezone, this timepoint could be anywhere in the world. Now let us localize the `datetime` object. We set the timezone to `US/Eastern`, and use the `localize` functionality from `pytz` to do the conversion. This date is in the summer, so daylight savings time is taken into account when deciding the offset in hours from UTC:

```python
>>> import pytz
>>> now_tzaware = pytz.timezone('US/Eastern').localize(now)
>>> print(now_tzaware)
2024-07-05 16:00:00-04:00
```

As you can see, the time is unchanged (still 16:00 or 4 pm) but now timezone information has been offset of -4 hours. This is useful when you have the local wall time and wish to add timezone information.

Next, let us convert this `datetime` object to UTC time. It is commonly an advantage to define times in the UTC timezone, and convert it to localtime if you need it. This is using a method of the `datetime` class, `astimezone()`:

```python
>>> now_utc = now_tzaware.astimezone(pytz.UTC)
>>> print(now_utc)
2024-07-05 20:00:00+00:00
```

Notice that the time is the same, but now in the UTC timezone, which is +4 hours offset with respect to the US/Eastern time zone, so the time is now given as 20:00 hours (8 PM).


### Daylight savings time {#daylight-savings-time}

The `pytz` module deals sensibly with daylight savings time. Let us define two dates, one in the winter and the other in the summer. These `datetime`  objects are initially without timezone information, this is often referred to as a "naive" object:

```python
>>> winter_naive = dt.datetime(2024, 2, 14, 12, 0)
>>> summer_naive = dt.datetime(2024, 7, 14, 12, 0)
>>> print("winter_naive:", winter_naive)
winter_naive: 2024-02-14 12:00:00
>>> print("summer_naive:", summer_naive)
summer_naive: 2024-07-14 12:00:00
```

Now let us make these `datetime` objects timezone aware, again we use the `localize()` function from `pytz` to achieve this:

```python
>>> summer_aware = pytz.timezone('US/Eastern').localize(summer_naive)
>>> winter_aware = pytz.timezone('US/Eastern').localize(winter_naive)
>>> print(summer_aware)
2024-07-14 12:00:00-04:00
>>> print(winter_naive)
2024-02-14 12:00:00-05:00
```

As you can see, due to daylight saving, the UTC offset is -4 hours in the winter and -5 in the summer.


### Beware of Local Mean Time (LMT) {#beware-of-local-mean-time--lmt}

A `pytz` timezone class does not represent a single offset from UTC, it represents a geographical area with an offset that varies. The average time offset for this area is called "Local Mean Time" (LMT) and is an offset from UTC by a number of minutes. If you do not use the `localize()` function from `pytz` you might get the LMT offset in the timezone aware `datetime` object. This is how that looks, first we save the `tzinfo` object in a variable `eastern`, and next use that to generate a timezone aware object, by directly attaching the `tzinfo` object:

```python
>>> eastern = pytz.timezone('US/Eastern')
>>> summer_aware2 = dt.datetime(2024, 7, 14, 12, 0, tzinfo=eastern)
>>> print(summer_aware2)
2024-07-14 12:00:00-04:56
```

As you can see, the UTC offset is now 4 hours and 56 minutes, and not a whole number of hours, and it does not take DST into account. This is evident if we convert to UTC:

```python
>>> print(summer_aware.astimezone(pytz.UTC))
2024-07-14 16:00:00+00:00
>>> print(summer_aware2.astimezone(pytz.UTC))
2024-07-14 16:56:00+00:00
```

Therefore, it is not recommended to define the timezone in this way, as it may be confusing. The representation of the `summer_aware` object has details about the LMT setting of this object:

```python
>>> print(repr(summer_aware2))
datetime.datetime(2024, 7, 14, 12, 0, tzinfo=<DstTzInfo 'US/Eastern' LMT-1 day, 19:04:00 STD>)
```


## The new way: The built-in `zoneinfo` module {#the-new-way-the-built-in-zoneinfo-module}

```python
import datetime as dt

def now_tzaware():
    # Return zone aware present time. This will also take care of
    # daylight savings time.
    return dt.datetime.now().astimezone()


# This is from the std library to
from zoneinfo import ZoneInfo

winter_naive = dt.datetime(2024, 2, 14, 12, 0)
print("winter_naive:", winter_naive)
summer_naive = dt.datetime(2024, 7, 14, 12, 0)
print("summer_naive:", summer_naive)

winter_aware = winter_naive.astimezone()
print("winter_aware:", winter_aware)
summer_aware = summer_naive.astimezone()
print("summer_aware:", summer_aware)

print("Convert to UTC")
print(winter_aware.astimezone(ZoneInfo('UTC')))
print(summer_aware.astimezone(ZoneInfo('UTC')))

print("Convert to US/Eastern")
print(winter_aware.astimezone(ZoneInfo('US/Eastern')))
print(summer_aware.astimezone(ZoneInfo('US/Eastern')))

just_before_dst = dt.datetime(2024,3,31,1,59).astimezone(ZoneInfo('localtime'))
just_after_dst = dt.datetime(2024,3,31,2,1).astimezone(ZoneInfo('localtime'))
print("1 minute before dst", just_before_dst)
print("1 minute after dst", just_after_dst)
print("difference", just_after_dst - just_before_dst, ", this is wrong!")


# Convert to UTC
delta =  just_after_dst.astimezone(ZoneInfo('UTC'))-just_before_dst.astimezone(ZoneInfo('UTC'))
print("convert to UTC:", delta)

```

----