+++
title = "Timezone Aware Timestamps in Python"
date = 2024-07-19T22:57:00+02:00
tags = ["datetime", "timezone", "python", "DST"]
categories = ["Python"]
draft = false
author = "Morten Kjeldgaard"
description = "Examples of how to use timezone aware datetimes in Python"
keywords = "python datetime pytz"
+++

The datetime object in Python is very powerful, but if you want to do serious timeseries analysis and use data from around the world, you need to use timezone aware `datetime` objects, and these are not the default.

<!--more-->

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

Since version 3.9, Python comes with the [ZoneInfo](https://docs.python.org/3/library/zoneinfo.html) module built-in. The `pytz` module we have used until now works well, but is a third party module that you need to install with `pip`.

Let's check out the ZoneInfo module, and especially how it behaves with daylight savings time. We'll declare a couple of dates, one in the winter, the other in the summer.

```python
>>> import datetime as dt
>>> from zoneinfo import ZoneInfo
>>> winter_naive = dt.datetime(2024, 2, 14, 12, 0)
>>> summer_naive = dt.datetime(2024, 7, 14, 12, 0)

>>> print("winter_naive:", winter_naive)
winter_naive: 2024-02-14 12:00:00

>>> print("summer_naive:", summer_naive)
summer_naive: 2024-07-14 12:00:00
```

The naive objects contain the date and time we expect. Now let's specify these according to the local timezone:

```python
>>> winter_aware = winter_naive.astimezone()
>>> summer_aware = summer_naive.astimezone()

>>> print("winter_aware:", winter_aware)
winter_aware: 2024-02-14 12:00:00+01:00

>>> print("summer_aware:", summer_aware)
summer_aware: 2024-07-14 12:00:00+02:00

```

The time at noon is unchanged, but the offset from UTC is specified. Notice that in this case, the timezone is +1 hour in the winter and +2 hours in the summer. Now let's convert these times to UTC. We now get to use the `ZoneInfo` module.

```python
>>> print("Convert to UTC")
>>> print(winter_aware.astimezone(ZoneInfo('UTC')))
>>> print(summer_aware.astimezone(ZoneInfo('UTC')))
Convert to UTC
2024-02-14 11:00:00+00:00
2024-07-14 10:00:00+00:00
```

These times are correct, also in the UTC timezone. Let's try converting to US/Eastern timezone:

```python
>>> print("Convert to US/Eastern")
>>> print(winter_aware.astimezone(ZoneInfo('US/Eastern')))
>>> print(summer_aware.astimezone(ZoneInfo('US/Eastern')))
Convert to US/Eastern
2024-02-14 06:00:00-05:00
2024-07-14 06:00:00-04:00
```

The times are still correct, the timestamp is the same, but of course the walltime is different in Europe and in Eastern US.

Now, let's do something that might , and that is taking time deltas across daylight savings time. We'll define two timezone aware timestamps, two minutes apart, at the time point where daylight savings time began in Europe in 2024.

```python
>>> just_before_dst = dt.datetime(2024,3,31,1,59).astimezone()
>>> just_after_dst = dt.datetime(2024,3,31,2,1).astimezone()
```

So, just after DST changes, the time is 02:01, right? Let's print out these objects and check:

```python
>>> print("1 minute before dst", just_before_dst)
>>> print("1 minute after dst", just_after_dst)
1 minute before dst 2024-03-31 01:59:00+01:00
1 minute after dst 2024-03-31 03:01:00+02:00
```

Notice that the timestamp immediately after DST begins is set at 03:01, which is correct. We tried to set the time to 02:01, but that timepoint simply doesn't exist. Fortunately the module corrected our mistake silently. Now lets print the difference in time between 01:59 before DST and 03:01 after DST:

```python
>>> print(just_after_dst - just_before_dst)
0:02:00
```

The result is two minutes, which is correct! Let's try to convert these to timestamps to UTC time and see what happens.

```python
# Convert to UTC
>>> just_before_dst_utc = just_before_dst.astimezone(ZoneInfo('UTC'))
>>> just_after_dst_utc = just_after_dst.astimezone(ZoneInfo('UTC'))
2024-03-31 00:59:00+00:00
2024-03-31 01:01:00+00:00
```

Now both timestamps have been converted to UTC, and the difference:

```python
 print(just_after_dst_utc-just_before_dst_utc)
0:02:00
```

is still 2 minutes.

So, it seems if you are using timezone aware `datetime` objects, things are pretty safe. However the topic of time is a complex one. The obvious answer, to always use timezone aware timestamps is however not always correct, since timezones change all the time, and DST may be abolished in the future. So a time marking is a volatile and subjective piece of data.

Joël Perras recently wrote a [blog post](https://nerderati.com/a-python-epoch-timestamp-timezone-trap/) on his site Nerderati describing the pitfalls you can encounter especially when using UNIX timestamps intermixed with Python `datetime` objects.
