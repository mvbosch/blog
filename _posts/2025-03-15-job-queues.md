---
title: "Job queues - let your webserver delegate busywork"
date: 2025-03-15
author: "Michael Bosch"
---

When your webserver starts receiving a lot of traffic and starts performing too slow, you have at least two options. You can throw more hardware (and, by extension, money) at the problem, which might aleviate the symptoms in the short term, or you can start optimising. In this context, the goal of optimisation is to reduce your server's request/response lifecycle to the shortest duration possible. Optimisation is an extensive subject and can in itself become a black hole of development time, especially if you are not experienced in profiling applications. In many cases that is necessary, but often we can improve performance simply by cutting out unnecessary work.

Let's say for example that your application allows users to login via a magic email link. With a naive approach, your server might be responsible for the following:

1. Query the data repository to verify that a user with the given email exists
2. Generate a token and attach it to the login url
3. Retrieve the HTML email template and replace placeholders
4. Initiate either a SMTP or HTTP request to an email service provider
5. Wait for a response from the email server
6. Respond with a success code

In this scenario, for security purposes, the server should _always_ respond with a success code even if no user record was found in step 1, as this guarantees no leaking of information to attackers. There is really no reason why this process cannot be asynchronous - there is no benefit to the user to process the request in realtime. Steps 2 through 5 can be processed by a separate worker process. We can then refactor the above to look like this instead:

1. Queue a job to send a magic link
2. Respond with a success code

Just like that, we've reduced the tasks that our hypothetical server needs to perform from 6 to 2, which by itself should guarantee faster response times. Since we're always responding with a success code, we can even offload the user verification step to the worker process. If no user is found, the job can simply terminate early and move on to the next task in the queue. Additionaly we can provide a better user experience with this pattern when everything doesn't work according to plan. Consider the possibility that the email server experiences an outage, how do we handle that with the synhronous approach? Our server can either respond with an error message asking the user to try again later or it can actively retry within the request/response lifecycle, _ultimately extending it_. With the worker process in play, we can retry the job with a backoff strategy all while the server happily processes additional requests and keeps the flow going.

In my experience, where jobs may fail, it pays off to break large jobs into smaller ones where possible. The reasoning for this is simple - you want to set sane timeouts and redo as little as possible on retries. [In my previous post]({% post_url 2024-03-08-other-benefits-to-modern-ssr %}), I wrote about generating ticket files from a template and posting them off to a customer. Initially I made the mistake of creating a job that generates all tickets and sends them off when done, which quickly caused some issues. First off, the worker framework I used had a default timeout of 10 seconds, which I left at that. Generating a ticket could take anything from 2 - 4 seconds, so jobs working on five or more tickets were failing. Okay, let's raise the timeout. What is a sane amount of time? 60 seconds? Wrong. We had records of customers purchasing more than 40 tickets at a time. So what then, no timeout? Oh boy, I did this and it was ugly. The ticket generator at some point delegated to a subprocess, which would rarely hang without writing any error message to the logs. Not only were tickets not generating, but the entire worker process was unresponsive, causing knock on effects to other parts of the system. This was in dire need of a refactoring.

I broke the job up into two parts. One job would now take responsibility for the generation of exactly one ticket file, while the other takes care of dispatching the tickets. This meant that I could set a timeout of sub 10 seconds, which is usually more than long enough to complete the task and also short enough to keep the queue going when things go wrong. The new architecture introduced a new problem though - we need the job execution order to be _somewhat deterministic_. We don't really care in which order the tickets get generated, but we don't want to run the dispatch job until _all_ of the tickets for an order are generated. Depending on the technologies involved, your flavour of job queue can follow a deterministic job execution order like [FIFO or LIFO](https://en.wikipedia.org/wiki/FIFO_(computing_and_electronics)) or it could be random. I opted to be safe and set a flag. With any luck, your worker process allows you to enqueue new tasks within a running job. 