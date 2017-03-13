---
layout: post
title:  "Time matters"
date:   2017-03-13 14:38:26 +0100
categories: programming time
---

> “I'm gonna kill myself. I should go to Paris and jump off the Eiffel Tower. I'll be dead.
> You know, in fact, if I get the Concorde, I could be dead three hours earlier, which would be perfect.
>
> Or w-wait a minute. It - with the time change, I could be alive for six hours in New York,
> but dead three hours in Paris. I could get things done and I could also be dead.”
>
> ― Woody Allen

There was written a lot about time. And if you are an experienced programmer, then you know that time is tricky.
And of course you've read a great article [Falsehoods programmers believe about time](http://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time),
might be even [More falsehoods programmers believe about time](http://infiniteundo.com/post/25509354022/more-falsehoods-programmers-believe-about-time).

So, as an experienced programmer you always use **UTC** to store time. And you are right!

# You are *almost* right ;)

UTC time is perfect for timestamps and timestamps are what an average programmer sees most of the _times_.

There is a nasty minority of cases when UTC time is just not enough.

The very good examples of those are:
 * Flight departure or arrival time.
 * Taxi pickup time.
 * Goods delivery time.

So what differentiates these from the others?
In all those examples three parts meet together:
 * Time of the _event_.
 * Place or location of the _event_.
 * Real humans.

Actually the latter is the cause of the trouble, as usually. Because those flawed humanoids use _local_ date/time,
and what are they interested in is not only _when_ something happens, but also _when_ it happens in the _time zone_ of the place where it happens.

And this is a big difference from an average timestamp.

Because if your bank transaction says "2017-03-12 14:00" when your browser in one time zone and
"2017-03-12 15:00" when you open your browser in different time zone, it's pretty normal. And understandable.

But when you see that your flight departure time you are interested in the **local time of the location** where boarding takes place.

Therefore, this post is about this special case and for the rest I encourage you to use UTC instant.

# The <sub>in</sub>famous local time

Ok, this is just a local time, can't we just keep time zone of the location next to UTC instant and transform it to the local
time when we need it? Isn't it just a presentation problem?

Yeah... You can do that.. but the place **where** you do transformation from UTC to local time is also important.

If we (over)simplify the use case, then usually what you have is

![My helpful picture]({{ site.url }}/assets/2017/03/time-problem.png)

Imagine that you have a web site which shows you a flight departure time, so if your API provides UTC instant and time zone,
then burden of transformation these to local time is problem of your user. And by user we mean the client of your API.
It might be a native application(desktop client or android application) or nice-looking web site. Thus transformation
of the date/time happens on the client side which you **don't control**.

Do you know how often the [Olson database](https://en.wikipedia.org/wiki/Tz_database) is updated?
Around ten<sub>ish</sub> times per year. Almost every month. Sometimes just before the actual change of the time zone takes place.
Sometimes _after_ the actual change, due to a late notice for example.

Do you know the version of that database which your browser use? On my laptop it is `2016j`, on my phone...
I'm not so sure really, I _hope_ that it's more or less fresh - at least it receives security patches occasionally,
so _might be_ manufacturer includes updates for `tzdata` package.

On my daughters tablet? Sure it's pretty old.

Do you dare the unknown device to do date/time transformation using unknown version of the Olson database(tzdata)?
If you do, then don't call it transformation anymore, name it mutation.

![cat-dog](https://i.imgur.com/PCLti9D.jpg)

Because results are unpredictable. There are good chances still that it will work correctly of course. Almost. Probably.

So, when you can't trust the client do so, then just don't do that.

**Use local date/time in your API in this case**.

Do date/time transformation on the server side, where you **can control** it.

And maintaining it is not an easy task as one can think:
 * First of all you have to subscribe to [tz-announce](https://mm.icann.org/mailman/listinfo/tz-announce) mail list, so you'll be up-to-date with changes.
 * You have to upgrade `tzdata` package in your operating system. Sometimes it's a good idea to notify maintainers about changes in Olson database.
 * If you use java, then you have to [upgrade built-in version of tzdata](http://www.oracle.com/technetwork/java/javase/tzupdater-readme-136440.html).
 * If you use joda-time, then [you are not saved as well](http://www.joda.org/joda-time/tz_update.html).
 * Your storage might use it's own copy of that database as well, for example PostgreSQL does. So you probably need to upgrade it.
 * It's a good idea to update that in all places at once.

# You are *almost* done ;)

![almost](https://i.imgur.com/3vgQCbh.jpg)

> You have delighted us long enough
>
> ― Jane Austen, Pride and Prejudice

Even if you use local date/time in your system's interfaces and keep transformations inside the system and of course
you have up-to-date `tzdata`, even then you might face some troubles. In rare cases, but that's possible.

What might happen:

 * Client provides some local date/time in the not so far future.
 * Server transforms this local time to the UTC instant and time_zone.
 * Then, there is a change in the time zone, for example DST starts earlier this year(yeah, that happens).
 * Next time when client asks for that date, it receives different local time, because transformation goes wrong.

So, you had everything right, but result is wrong. Life is unfair. Deal with it.

# What you _can_ do

You can use local date/time everywhere. If there is no need to show that time in time zone different from where
_event_ takes place then you don't need to transform it.
And you don't need to build a service which returns you a time zone
based on location(like [that on](https://developers.google.com/maps/documentation/timezone/intro)).
And do you _always_ know the exact location?

In this case your date/time will be consistent all over your system and the chance of mistake will be minimal.

That's what you should strive for.

# When you **do need** to transform local time to UTC

Probably the only case when you do need to transform local time to UTC instant is when
you need to trigger time-based events.

For example to send notification one day or an hour in advance. Or to schedule some processing task.

That's the moment then you should breath in and introduce date/time transformation into your system.

And now you have two choices:

* You can have local date/time as your primary date structure and
do transformation only when you have to.
* You can transform local date/time to UTC instant+time zone as soon as it crosses the boundaries
of your system and transform it back when it's requested.

I personally prefer first approach as there are less chances to make a mistake due to the reasons described above.

But some developers prefer to have UTC instant as primary date structure and convert it every time it's requested.

Don't make that choice blindly - look closely on your use cases.
It can be that one way is more convenient for your system than another.

