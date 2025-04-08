---
title: "Preventing order collisions with real-time updates"
date: 2025-04-07
author: "Michael Bosch"
---

One of the biggest challenges we face in ecommerce is order collisions - when two or more customers checkout the last available item in stock. Learn how we can overcome them with real-time updates and a layered approach.

We built a web application that sells tickets for two types of events:
* Events with general admission where ticket sales are only limited to attendance capacity. The only requirement is not to oversell.
* Events with a seat allocation or floor plan. Like in a movie theater, each seat is unique and we can't expect customers to pile atop one another (unless the show is _really_ good). For the UI, we made an interactive seat map using SVGs.

We need to show customers which tickets are available, or in the case of general admission events, how many tickets are available. Naturally you'd retrieve this information on page load, but you can't stop there. Users view your page as the source of truth, though in reality the database is the source of truth. If the user's page runs out of sync with the database, they're set up for a bad experience. Also, if you're under the impression that customers always load your page and immediately purchase what they want, I have bad news for you. People will open your page on a Tuesday and expect it to still show relevant information in 2058. 

To manage page state, we can talk about polling, Server Sent Events (SSE) and the weather but I'm really here to tell you the good news about websockets. Websockets give web application servers the ability to push data to subscribed browsers, which can be used to update the UI in near real-time[[1]](#1). SSE can also facilitate this but has restrictions on connection count, so it gets the back seat in this discussion. With websockets, we can define any number of "channels" and subscribe clients to relevant ones where they'll receive broadcast messages. Think of it as a group chat with a specific audience, except in this context clients don't get to message one another directly.

Let's look at an example where both customers X and Y want to purchase tickets for an event named `Breakfast: A Sequel Called Lunch`:
1. X and Y visit the event page, which is loaded with the current seat/ticket statuses. The floor plan is rendered with unavailable seats made noninteractive. Both clients connect to the websocket endpoint, e.g. `wss://example.com/events/breakfast-ii/messages`
2. Customer X selects seat `A1`
3. Customer X confirms their selection and the server is requested to add the seat to their cart (without confirmation, indecisive customers cause unwanted traffic and state management)
4. The server:
   1. Confirms that seat `A1` is in fact available.
   2. Adds the seat to customer X's cart and broadcasts a message to this event's websocket channel.
5. Finally, customer Y's browser receives the message and marks seat `A1` on the SVG as `unavailable`.

Nice! Is it foolproof? Not quite yet, but it helps keep the majority of collisions at bay. In order to ensure that things run smoothly, seat allocation is again verified both when initiating a payment and once payment has been confirmed. Having a layered solution is the best defence - so make use of all your available tools. Check that your DBMS supports locking rows on read with `SELECT FOR UPDATE` or similar and use this to your advantage, ensuring that your `ecommerce_product` record remains locked for the duration of each transaction.

So what about abandoned carts? Customers are granted a limited window of opportunity to checkout their order and secure their selection. Background tasks set to trigger at specific times ensure that abandoned seats are once again allocated to the available pool.

Pro tip: keep your UI as dumb as possible and provide **absolute** quantities when it needs to update, i.e. `{"available": 10}`. Don't make the mistake of providing relative quantities like `{"sold": 3}` and expect the UI to figure it out. Your tears will fill oceans.

<span id="1">[1]</span> - Yes, "real-time" has a specific meaning in the domain of computer science. I am using the term in the context of human perception.
