# Payment Providers - Recurly

Recurly specializes in managing recurrent billing including subscriptions, plans, and the ability to change plans mid-subscription.

## Cancellation

In Recurly, cancelling a subscription is only a cancel-request. The subscription in question will be marked for cancellation, which simply means that the consumer will no longer be
charged for the next subscription at the current subscription's termination date. The subscription will still be in the **active** state even when marked cancel-requested upto the
day before termination date.  On the termination date and after, the subscription is considered **expired**.

## Terminating

Recurly makes a distinction between cancelling and terminating subscription. Cancelling just means marking it not to recur at the termination date. Terminating will set the termination
date immediately, or *now.*

However, there is a big catch. There is *NO API CALL* to perform this action. It must be done administratively in the Admin Console. Refunding for unused time will be an option that
must be explicitly chosen when we perform this.

## Upgrading and Downgrading

Upgrading and downgrading are refered to as subscription changes in Recurly. Fortunately, since it's straightforward, we may immediately change the contract *at any time*. And Recurly
will automatically calculate credits for the unused portion of the older plan and charges for the new plan (unit (second) price of the new plan times how many days left).

For example, let's say that the user is currently on the Basic plan, $50/month billed every month. They signed up on January 1, and it is set to recur on February 1. They want to
upgrade up to the Expert plan, which is $80/month also billed every month. They want to upgrade on January 7, or 25% of the way (approx for simplicity of calculation), which will 
result in 75% unused time for the old contract. So, first, let's take care of the old contract. They should be a credit of $37.50 for the unused portion of their current plan.
For the higher plan, they'll be charged $80.00 * 0.75 (the remaining unused portion from the old contract) or $60.00. So the net charge to the customer will be $60.00 - $37.50 =
$22.50.

A similar calculation is performed for a downgrade, only resulting in a net refund to the consumer.

However, we may choose to upgrading immediately and downgrade-request only to avoid refunding.

## Developer Notes

Personally, I would refrain from using Recurly for only one reason - the returned body *has to be* XML, which is a pain to develop in, especially in .NET Core, which readily accepts
JSON. There is a .NET client for Recurly, which will do all the heavy XML-lifting for us, but that .NET Client is *not* compatible with .NET Core.


