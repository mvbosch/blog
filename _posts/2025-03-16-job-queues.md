---
title: "Don't make HTTP clients wait, delegate to job queues"
date: 2025-03-15
author: "Michael Bosch"
---

Sometimes webservers start performing poorly under load because of bad code practices or inefficient database queries. In that case you'd ultimately need to refactor and optimise, but sometimes we can achieve cheap gains by identifying areas where the server is doing work that it could otherwise delegate.

Let's take for example an application that allows users to login via a magic link sent via email. With a naive approach, the server might be responsible for the following:

1. Query the data repository to verify that a user with the given email exists
2. Generate an authentication token
3. Retrieve the HTML email template and replace placeholders
4. Initiate either a SMTP or HTTP request to an email service provider
5. Wait for a response from the email server
6. Respond with a success code

In order to avoid leaking user's emails to potential adversaries, the server should always respond with a success code, even if no user data could be retrieved in the first step. With that in mind, here's how we could refactor the above using a worker process:

1. Queue a job to generate a link and send an email
2. Respond with a success code

Take note that we can even offload the user verification step to the worker process. If no user is found, the job can simply terminate early and move on to the next task in the queue. Not only does the server get more breathing room to handle requests, we can also provide a better user experience when exceptions are encountered. If the email server experiences an outage, how do we handle that in our first example? Our server can either respond with an error message asking the user to try again later or it can actively retry within the request/response lifecycle, _making it even busier than it was before_. When delegating to the worker process, the job can happily retry with a backoff strategy in the background while the server remains available to process additional requests.

This pattern is applicable to many scenarios, it's a question of whether the result of the task being executed has any influence on the immediate server response. Do you make HTTP clients wait while your server encodes video they've uploaded? Delegate. Is your payment processor's webhook notification waiting while your server updates an order? Delegate. I promise you that your payment processor only cares that you acknowledge the message, not how you react to it internally.

In my experience, a bunch of smaller jobs with single responsibility is more manageable than a monolithic approach. The reason is simple - you want to set sane timeouts for when jobs run into trouble and redo as little work as possible on retries. [In my previous post]({% post_url 2025-03-08-other-benefits-to-modern-ssr %}), I wrote about generating ticket files from a template and posting them off to a customer. I made the mistake of creating a single job that generates all tickets files at once _and_ sends them afterward. Generating one ticket could take up to four seconds, so jobs often timed out after the default of ten seconds. My initial "fix" was to increase the timeout, but I had trouble figuring out what the exact value should be. We had records of customers purchasing more than forty tickets at a time, but what if someone ordered sixty? It's tempting to think along the lines of "_surely no one will order a hundred..._" and use that as a baseline. I was wise enough to resist the urge of predicting user behaviour, but stupid enough to set an **infinite** timeout. A part of the ticket generation code would call a subprocess, which on rare occasion would hang without writing any messages to the log, proving a challenge to debug. This behaviour led to the worker process being unresponsive as a whole and unable to process _any_ jobs. The pattern was in dire need of a rework.

I broke that job up into two parts. `generate_ticket` is responsible for generating exactly one ticket, while `send_tickets` is self explanatory. Since I know the average ticket generation time, I could set a timeout with some leeway or just stick with the default. Much better. The new design introduced a new problem though - I need the job execution order to be _somewhat deterministic_. It doesn't matter in which order the tickets get generated, but of course `send_tickets` shouldn't be invoked until _all_ of the tickets for an order are generated. Depending on the technologies involved, your flavour of job queue can follow a deterministic job execution order like [FIFO or LIFO](https://en.wikipedia.org/wiki/FIFO_(computing_and_electronics)) or it could be random. I use [saq](https://github.com/tobymao/saq) which has bad documentation but works pretty well. I opted to be safe and added a boolean parameter to the generation job, which when true, will iterate over a given order's ticket records and verify that all have a file location set, then enqueue the `send_tickets` job.

Worker processes introduce additional complexity which adds to your project's maintenance overhead, but they're often critical if you have scaling needs. Centralised logging and other observability tools can go a long way to ease the burden.
