---
title: "Simplify checkout, increase conversion rate"
date: 2025-03-30
author: "Michael Bosch"
---

A bumpy checkout process will have a drastic effect on your bottom line. I've heard many variations of this concept before, but it was sobering to see the results with my own eyes. Our client experienced more than a 300% increase in sales by removing friction in the checkout process.

Three years ago I was part of the development team that built an onboarding site for a monthly subscription based product. The client had negotiated a deal with a payment processor beforehand, so that integration was beyond our control. They decided to go with a debit order system which is pretty laborious - customers had to verify OTPs and the system ran various validations behind the scenes to verify bank account particulars. This makes for a very robust system from a fraud risk perspective, though at the cost of user experience. Some of these validations could take up to thirty seconds to complete and customers had to step through five forms to complete the checkout - the leanest we could make it. We had a successful launch and customers were buying. As with any sales system, there were indications of customer drop-off at various points in the process, though this was deemed to be within expected margins.

The client recently decided to switch to a different payment provider and use a much simpler debit/credit card payment system. We could do away with the OTPs and reduce the required form fields to the bare minimum personal information, followed by an embed of the payment processor's form, so effectively a two step process. We were very happy with the result because not only is the user experience much better, the resulting application is much easier to maintain as well.

Sales tripled overnight. That is _huge_. The mind blowing part is that there was zero marketing involved, we launched the new site to the existing domain. Not only that, the client's product had not undergone changes either. The payment process had been the _only_ deciding factor.

Oddly enough, we kept the old site running and provided a link for users who might prefer to pay via debit order and about 15% of conversions still came through this path. Clearly it pays to have options even if some of them seem inferior.

I tend to be a bit stubborn, so personally I can put up with a moderate amount of friction if I really want something. This experience really enlightened me not to project that way of thinking to others at large. Users in general will follow the path of least resistance, even if that path leads to other suppliers.

It goes to show that effective software is more than just efficient algorithms. No one cares that your server is fast if it's fronted by a poor interface. Whether directly or indirectly, the purpose of software is to serve people and enrich their lives. We need to look at software holistically and ensure a good experience throughout the entire stack.
