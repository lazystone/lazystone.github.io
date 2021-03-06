---
layout: post
title:  "And again about time"
date:   2017-03-29 13:28:00 +0100
categories: programming time java
---

![B2F]({{ site.url }}/assets/2017/03/b2f.jpg)

If you think that [February 30](https://en.wikipedia.org/wiki/February_30#Swedish_calendar) is invalid date or a town is always in [one timezone](http://www.cbsnews.com/news/town-in-two-time-zones/2017-03-29-one-time-more.md),
then read about other [FalsehoodsAboutTime.com](http://FalsehoodsAboutTime.com) and count how many do _you_ have.

Date and time is incredible complex subject when you work or at least aim to work world wide.
In order to help you to cope with that complexity you have to learn your 'enemy'.

# What kind of time is it?

![WTIT]({{ site.url }}/assets/2017/03/whattimeisit.jpg)

Whenever you see a date or time in the code or is about to introduce a new one, stop for a moment.
And use this moment to answer one question: What does this time actually represent?

In order to answer this question you should understand who is a consumer of that time and how this time will be used.

There are at least two kinds of time: timestamps and local times.

## Timestamps

That's probably 99% of all dates and times in your system.
And general rule here is to keep them as UTC instants.

[Timestamps](https://en.wikipedia.org/wiki/Timestamp) are usually generated by the system and if they ever consumed by the real human,
then they should be transformed to the local date time of the time zone where that human is located right now.

The most common examples are logging or audit events: create at, updated at, disabled at.
Note that all those times represent something in the past. Something what already has happened.

Another example is future event which is generated and consumed by a system(not a human), for example
scheduling some action: `reportToBeSentAt = someTimeInTheFuture.minusHours(2)`.

## Dramatic construction or the 1% of local times

> The three principles of dramatic construction derived by French neoclassicists from Aristotle's Poetics,
holding that a play should have one unified plot (unity of action)
and that all the action should occur within one day (unity of time)
and be limited to a single locale (unity of place).

If we rephrase this to something simple, then we get three ingredients describing some event:
* Time of the event.
* Place of the event. Which is so important as the time.
* Real humans must act in the event.

Instead of humans it might be a 3rd-party system which produces or receives date/time in local time format.

The examples are:
* Any kind of transportation. Flight departure times, taxi pickup time and etc.
These date/times presented to the human in the time zone where event takes place.
It's a rare and probably non standard case when those times should be shown in the different time zone instead.
* Appointments. This is quite similar to the previous case.
* Start or end of some period. Those even might have different UTC instant depending on the place.
If some discount ends on March 31 23:59:59, then consider that March 31 ends not at the same time around the world.

And the general rule for this case is to **store** date/time of the event as local date/time in the time zone
where event takes place.
The word 'store' is stressed out because it's important what you use as date/time source of truth.
Of course you might transform your local date/time to UTC instant and back when you need it,
but always keep the original local date/time representation.

Take a note that those three ingredients mentioned above are very important.
If you remove just one, then probably you'd better to use UTC instants instead:
* No time? No problem :)
* Real humans are not involved? Then use UTC instant.
* Place is not important or event covers several places(for example meeting over internet),
then you'll be good with UTC instant as well.

## Why not to store local time as UTC instant plus time zone?

Actually you can, but this has some drawbacks.

* Burden of transformation:
  * If date/time is produced as local date/time and consumed as local date/time then you might not need to transform it at all.
  * Sometimes it's hard to get a time zone of the event where event takes place.
    For example if person from India is booking a taxi in Los Angeles, then you need to know
    the time zone in Los Angeles at the moment of producing and presenting that UTC instant.
  * The time zone database(`tzdata` or [Olson database](https://en.wikipedia.org/wiki/Tz_database))
    might be outdated. Especially when it's not shipped as a part of software, but rather provided by operating system  .
* Risk of time zone database to be changed _after_ you created a time in your system, but _before_ that time is consumed.
  It might be seen as a really special case, but unfortunately it's not:
  Olson database is being changed and updated more than ten times during the year usually.
  So, in average every month some country decides to stop or start DST earlier, or cancels it,
  or some town gets it's own time zone, or change old one to another one.

In order to avoid those problems follow simple rules:

* Always keep original local time.
* Keep date/time transformations inside the system.
  If you are dealing with local times then don't demand from your API users to  provide UTC instant instead.
  By doing this you loose original meaning of that time.
  In addition you put additional responsibility on the client to know the proper time zone and to have
  up-to-date version of the time zone database.

  
# Date and time in Java

* Avoid `java.util.Date`. Period.
* Get familiar with [java.time](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html) package and learn how to use it.
  It has neat support for almost all use cases and works as almost drop-in replacement of JodaTime.
* Avoid 'default' time zone when converting to or from local date/time.
  If you don't know where to get a time zone, then it's a good sign of that something is missing in your system's design.
* Don't use `java.time.ZonedDateTime` to store information about your UTC instant.
  Always use data type which reflects the original value of the timestamp:
  >ZonedDateTime holds state equivalent to three separate objects,
  a LocalDateTime, a ZoneId and the resolved ZoneOffset.
  
  That means that if your original UTC instant points to somewhere in the magical double hour when DST ends
  and clocks are turned backwards, then that instant will change, if it points to _after_ the end of DST.

If you deal with local date/time, then it might be a good idea to store original time as
`java.time.LocalDateTime` and next to it store `java.time.Instant` which always can be recalculated
in case when time zone of the event changes. This will be needed only before that event actually happens.
 
# Maintenance

As it mentioned before time zone database changes quite often.
Therefore in order to reflect those changes in your system you have to apply some efforts.

## Java

 * Subscribe to [tz-announce](https://mm.icann.org/mailman/listinfo/tz-announce) mail list,
   so you'll be up-to-date with the changes.
 * Always keep `tzdata` package(s) of the OS up-to-date.
 * For Oracle JDK you have to [upgrade built-in version of tzdata](http://www.oracle.com/technetwork/java/javase/tzupdater-readme-136440.html) using tzupdater tool.

## Android

If you use joda-time, then
 * Subscribe to [tz-announce](https://mm.icann.org/mailman/listinfo/tz-announce) mail list.
 * You have to fork original joda-time. Follow the [official instructions](http://www.joda.org/joda-time/tz_update.html) for more details.
 * The same applies to android specific fork of [joda-time](https://github.com/dlew/joda-time-android).
 
## Javascript

 * If you use something like [Moment Timezone](https://momentjs.com/timezone/), then keep it up-to-date in your project.

