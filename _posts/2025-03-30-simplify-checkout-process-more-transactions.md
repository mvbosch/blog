---
title: "Simplify checkout, process more transactions"
date: 2025-03-30
author: "Michael Bosch"
---

An bumpy checkout process will have a drastic effect on your bottom line. I've heard many variations of this line, but it was sobering to see the results with my own eyes. Our client experienced more than a 300% increase in sales by removing friction in the checkout process.

Three years ago I as part of the development team that built an onboarding site for a monthly subscription based product. The client had negotiated a deal with a payment processor beforehand, so that part was beyond our control. They decided to go with a debit order system which is pretty laborious - customers had to verify OTPs and various validations were run behind the scenes to verify bank account particulars. This makes for a very secure system from a fraud risk perspective, though at the cost of customer experience. Some of these validations could take up to thirty seconds to complete! Customers had to step through five forms to complete their signup - the leanest we could make it. We had a successful launch and customers started buying in. As with any sales system, there were indications of customer drop-off at various points in the process, though this was deemed to be within expected margins.

A couple of weeks ago, the client decided to switch to a different payment provider and use a much simpler debit/credit card payment system. We could do away with the OTPs and reduce the required form fields to the bare minimum personal information, followed by an embed of the payment processor's card information form, so effectively a two step process. We were very happy with the result because not only is the user experience much better, the resulting application is much easier to maintain as well. I'm curious whether the new system results in more chargebacks due to fraudulent actors, but I have no data on that.

I've already mentioned the result, a 3x increase in sales. That is _huge_. The mind blowing part is that there was zero marketing involved, we launched the new site to the existing domain. Not only that, the product was unchanged as well. The payment process had been the _only_ deciding factor.

I tend to be a bit stubborn, so personally I can put up with a moderate amount of friction if I really want something. This experience really enlightened me not to project that way of thinking to everybody. It's a big world and there's always an alternative to your offering. People might even resort to an inferior product if it's easier to get!

As developers, it's easy to focus on the performance of our code, but software is more than just efficient algorithms. Software is ultimately used by people, whether directly or indirectly, and how your software is presented is crucial. People don't care that your software is faster if they don't feel good using it.
