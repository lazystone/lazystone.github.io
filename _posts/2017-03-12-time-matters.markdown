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

**Almost**.

UTC time is perfect for timestamps and timestamps are what an average programmer sees most of the times.

There is a nasty minority of cases when UTC time is not enough, or sometimes it just doesn't fit the use case.
It's when not only time but place also matters.

The very good examples are:
 - Flight departure date.
 - Taxi pickup time.
 - Goods delivery time.

So what is differentiate these from the other _times_?
In all those examples three parts meet together:
 - Time of the _event_.
 - Place or location of the _event_.
 - Real humans.

Actually the latter is the cause of the trouble, as usually.