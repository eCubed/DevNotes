# Payment Providers - Primer

One required skill for developers often ommitted academically and in even advanced developer courses and boot camps is the ability to take customers' payments. There are
many things to consider with customer payments, including ensuring that customers' data and transactions are secured. We will also need to know how to void and refund
payments. We will need to know how to set up recurrent billing and make decisions on how to go about upgrading, downgrading, and cancelling plans. Many of the mentioned
tasks are offered by payment providers. We sign up to a payment provider. They give us a sandbox account to test payments, refunds, etc. They provide us with API documentation
on how to charge a customer, check an account's status, refunding, voiding, as well as other queries such as how much we've earned over a certain period of time. Though
they offer us all of those services, each of them may handle things differently especially when managing plan changes. For example, some won't allow plan change until the
current contract is over while some do. Beyond what payment providers offer, we still need to decide what data we store on our end. We already know that we will *never* store
sensitive customer information in our system, not even in encrypted form. We would also need to decide *under which circumstances* some financial data we decided to store
can be deleted.

So, what is the purpose of this series? It is to compare how several payment providers operate, especially how they handle subscriptions, plans, and plan changes. We will not
address one-time payments as those are considered trivial for our purposes. So, here are the list of tasks we seek to accomplish with all payment providers.

1. **Store to and retrieve from the payment provider an account associated with our user.** Since we will not store any sensitive customer data, the payment provider will keep
those records in a secure manner. When we first create the account, we should *receive* a *unique identifier* for that account so that we may associate it with our user, thus
store it *on our end*. This way, at a later time, we will be able use the account's unique identifier to make subsequent charges at will (if we decide that we will initiate
billing ourselves instead of letting their subscriptions service manage recurring charges).
2. **Make a payment, and receive the reference for that payment.** There are different ways to make a payment, though for all of them, we know we would report at least the
items purchased, their individual prices, and who we're charging. Regarding *who* we're charging, some payment API calls will require the customer's personal and financial
information. Some payment API calls will only ask for a customer's reference number (that we would have already stored in *our* system) along with what we're charging them.
Regardles of the API call made to make a payment, we should receive a reference number (usually alpha-numeric) from that transaction. Now the tricky part is deciding whether
to store that reference number on our end. It will always be available from the payment provider later if we'll need it.
3. **Voiding vs. Refunding.** From a purely technical standpoint, these are just API calls that perform two separate, but subtly different transactions. Of course, we may
need to provide the payment reference number to know which payment to void or refund. We may need to know the difference between the two so that we may understand the error
messages when we attempt to void or refund. We can only void payments that have just been made. The successful response from a payment just means that the customer's info
are correct and they have enough money in their account to make a payment. At this point, their account has *not yet* paid the merchant. The transaction right now, though
successful, is still considered *unsettled* since money has *not* yet been officially transferred. Later, depending on payment provider and the customer's card or financial
institution, money has already been transfered to the merchant. Now, the transaction is considered *settled*, and when settled, we can only refund, not void a payment. Some
payment providers make it easy for us - when we decide to no longer charge the customer, they will attempt a void first, and if that wasn't successfull (because the
transaction was already settled), they will make a refund.
4. **Plans, Subscriptions, and Recurrent Billing.** There is a lot to discuss about plans, subscriptions, and recurrent billing. A plan is a combination of how much we want
to charge the customer for a specific duration of time. A subscription is which plan a customer has at any given time, which consequently contains info of when the subscription
started and when it should end. Recurrence is a decision of whether to automatically charge the user at the end of the current subscription for a new subscription. Though
many payment providers allow us to set these up with customers, we still need to decide which reference numbers to store on our end, and how we'll associate them with our
own entities.
5. **Cancelling a Subscription.** This is the easiest type of plan change, though payment providers may handle this differently. When a user wants to cancel their current
subscription, some payment providers will only mark the subscription to *eventually cancel* at its termination date. In this case, the customer's subscription is still
considered active *after* they notified us that they want to cancel. Sometimes, we may want to *immediately cancel* the user's subscription. This means that the unused
portion of their subscription must be refunded to the user immediately, and the subscription's status should no longer be active. This way, when we need to query their
current subscription, we would know at the spot that we can deny them certain services. When selecting a payment provider, we need to know how they handle subscriptions
and if they provide options to immediately cancel.
6. **Upgrading mid-Subscription.** Many times, a user may want to upgrade their current subscription, and they're in the middle of a subscription. Some payment providers will
not allow, at least by default, immediate upgrading, which means they would only mark the current subscription as "upgrade to the new plan at the end of this subscription".
That's the easiest execution. However, users may want immediate upgrading, and this is where things get very tricky because some payment providers don't allow it, and some
payment providers handle the financial transactions differently. It seems like when we immediately upgrade a customer, they will should pay right now the cost of the new plan
minus the unused portion of their current subscription. They should get a new subscription that is in effect immediately, and they get a new billing cycle whose effective and
termination dates are different than their then-current subscription. Some payment providers may refund the user for the unused time first, but charge the full price for the
new contract. Sometimes, while the customer may get a refund of the unused portion of the then-current subscription, they will be billed the new plan immediately *but* only
for the remaining time of the then-current contract. This kind of fix will preserve the billing cycle. When surveying payment providers' plans and subscriptions, know how
their proration works because it may be different from what we expect.
7. **Downgrading mid-Subscription.** This is quite similar to the concerns of upgrading mid-subscription. However, the trick is when the current subscription's unused money
exceeds the lower plan's charge, we may need to refund the difference should we decide (provided that the payment provider allows) to immediately downgrade an account.
8. **Notification of recurring charge success or failure.** When a subscription is set to recur at the end of its period, we would need to be notified. This means that we
would need to provide the payment provider with a url that *they* can call (web hooks). Typically, this is an API web method that we would need to write. When the payment 
provider makes the call, the returned object must include the customer's reference number so that we can match it with the correct user. As a client of the payment provider,
we would need to make a decision whether we would want to be notified of financial events. If we decide that we would check with the payment provider (the status of the 
current contract) before every user action, then we may not need to set up web hooks.