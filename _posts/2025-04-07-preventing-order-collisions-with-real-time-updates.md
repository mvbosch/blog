---
title: "Preventing order collisions with real-time updates"
date: 2025-04-07
author: "Michael Bosch"
---

One of the biggest challenges we face in ecommerce is order collisions - when two or more customers checkout the last item you have in stock. Learn how we can overcome them with real-time updates and a layered approach.

We built a system that sells tickets for two types of events:
* Events with general admission where ticket sales are only limited to attendance capacity. The only requirement is not to oversell.
* Events with a seat allocation or floor plan. Like in a movie theater, each seat is unique and we can't expect customers to pile atop one another (unless the show is _really_ good).

We need to show customers which tickets are available, or in the case of general admission events, how many tickets are available. Naturally you'd retrieve this information on page load, but you can't stop there. Users view your page as the source of truth, though developers understand that the database is the source of truth. If these numbers run out of sync, users are set up for a bad experience. Also if you're under the impression that customers always load your page and immediately purchase what they want, I have bad news for you. People will open your page on a Tuesday and expect it to still show relevant information in 2058 (hello, future!). 

Now we can talk about polling, Server Sent Events (SSE) and the weather but I'm really here to tell you the good news about websockets. Websockets give web application servers the ability to push data to client browsers, which can be used to update the UI in near real-time. SSE can also facilitate this but has restrictions on connection count, so it gets the back seat in this discussion.

Pro tip: keep your UI as dumb as possible, don't make it do math. If you're updating available quantities, just let the server send the truth as a whole, i.e. `{"available": 10}` and avoid trying to track state with subtractions and other nonsense. The backend is the authoritive source and should do **all the math**.
