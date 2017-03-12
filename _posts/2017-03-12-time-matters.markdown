---
layout: post
title:  "Time matters"
date:   2017-03-12 17:38:26 +0100
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
 - Flight departure or arrival time.
 - Taxi pickup time.
 - Goods delivery time.

So what differentiates these from the others?
In all those examples three parts meet together:
 - Time of the _event_.
 - Place or location of the _event_.
 - Real humans.

Actually the latter is the cause of the trouble, as usually. Because those flawed humanoids use _local_ date/time,
and what are they interested in is not only _when_ something happens, but also _when_ it happens in the _time zone_ of the place where it happens.

And this is a big difference from an average timestamp.

Because if your bank transaction says "2017-03-12 14:00" when your browser in one time zone and
"2017-03-12 15:00" when you open your browser in different time zone, it's pretty normal. And understandable.

But when you see that your flight departure time you are interested in the **local time of the location** where boarding takes place.

# The <sub>in</sub>famous local time

Ok, this is just a local time, can't we just keep time zone of the location next to UTC instant and transform it to the local
time when we need it? Isn't it just a presentation problem?

Yeah... You can do that.. but the place **where** you do the transformation from UTC to local time is also matters.

If we (over)simplify picture, then usually what you have is

![My helpful picture]({{ site.url }}/assets/2017/03/time-problem.png)

Imagine that you have a web site which shows you a flight departure time, so if your API provides UTC instant and time zone,
then burden of transforming these to local time is problem of your user. And by user we mean the client of your API.
