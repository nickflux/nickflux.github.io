---
layout: post
title: Has your worker finished yet?
category: posts
---

> “Does he have a pulse?”
> 
> “Rather! His pockets are full of lentils.”
— What Ho! Cabbie

At [Givey][1] we use [Sidekiq][2] for our asynchronous job processing. It's great, but it does have a mysterious, fire-and-forget quality to it which can sometimes feel a bit fire-and-oh-yes-did-I-not-tell-you-that-job-did-not-finish-at-all. That is not a good feeling to have. I'd prefer to have the worker tell me if it didn't complete, preferably within a given timeframe.
Tricky for the worker to do, of course. The failure might be due to an external service, so you need to keep open the possibility of re-tries. And if the worker suffers an internal complication the only thing you are likely to hear are its dying words, croaked into [Honeybadger][3].

On top of that you have the problem of chaining together multiple jobs. Two of the most common tasks for our workers are processing donations (using the PayPal API) and sending emails — obvious jobs for asynchrony — and usually we want to process a donation and then send the email confirming the donation has been completed, or not. So should we have one job finish and fire off another job, compounding the problems mentioned above? Or, should we have one job do both, totally different tasks, thereby nauseating [Gary Bernhardt][4] acolytes?

What we need is something to wrap round these jobs to monitor their forward progress and rat them out when they let us down.
We haven't done that yet, but we have implemented a very simple solution to check the heartbeat of Sidekiq and Redis which suggests a possible solution. Every few minutes a cron job chucks an invalid Twitter donation into our API. A donation record is created and a worker starts processing it. All that involves is changing its status from _pending_ to _invalid_. We have a page that [Pingdom][5] checks once a minute to see if the most recent donation of this type is _invalid_ — that means it has been processed. This is not an elegant solution but it has the benefit of fitting into our existing alert system, and it shows how you can externally monitor a job: wait a bit and then check to see if a database record has the expected completion state.

The next step for us is to create a work flow type process that can fire off one or more jobs in series, wait to see if they complete successfully and report back if not. Of course, then you have the problem of monitoring the work flows, but even our greatest philosophers had [trouble with that one][6].

[1]:  http://givey.com
[2]:  http://sidekiq.org/
[3]:  https://www.honeybadger.io/
[4]:  https://twitter.com/garybernhardt
[5]:  https://www.pingdom.com
[6]:  http://www.goodreads.com/quotes/472433-oh-the-jobs-people-work-at-out-west-near-hawtch-hawtch
