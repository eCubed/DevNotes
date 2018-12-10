# Payment Providers - Stripe

Stripe is one of the newer payment providers whose focus is more on clean REST APIs.

## Cancellation

It is possible to immediately cancel the subscription at any time during mid-subscription. The subscription *will be automatically be given* the **cancelled** status. The subscription
will no longer recur. By default, unused time (until the normal termination date of the contract) will be credit back to the user *at the normal termination date of the contract*!
So, the financial transaction for cancellation is NOT immediate. 

There is an `invoice_now` argument to the cancel request, which appears to be a boolean value. The documentation states that it (presumably if set to true) "will generate a final
invoice that invoices for any un-invoiced metered usage and new/pending proration invoice items." My impression is that generating the invoice for these kinds of transactions means
that the customer will be notified about the cancellation *and* will be refunded *immediately* for unused time, NOT at the end of the normal termination date.

## Upgrading and Downgrading

Upgrading and downgrading are refered to as subscription changes in Recurly. Fortunately, since it's straightforward, we may immediately change the contract *at any time*. And Recurly
will automatically calculate credits for the unused portion of the older plan and charges for the new plan (unit (second) price of the new plan times how many days left).

For example, let's say that the user is currently on the Basic plan, $50/month billed every month. They signed up on January 1, and it is set to recur on February 1. They want to
upgrade up to the Expert plan, which is $80/month also billed every month. They want to upgrade on January 7, or 25% of the way (approx for simplicity of calculation), which will 
result in 75% unused time for the old contract. So, first, let's take care of the old contract. They should be a credit of $37.50 for the unused portion of their current plan.
For the higher plan, they'll be charged $80.00 * 0.75 (the remaining unused portion from the old contract) or $60.00. So the net charge to the customer will be $60.00 - $37.50 =
$22.50. However, this $22.50 is not charged immediately. It will be charged at the end of the billing cycle along with the charge for the new plan in full. They will then be charged
$22.50 + $80.00, which is $112.50, at the end of the current billing period.

A similar calculation is performed for a downgrade, only resulting in a net refund to the consumer.

If we want to collect or refund the net amount from upgrading and downgrading *immediately*, it looks like we will need to generate an invoice whose `billing` field is set to
`charge_automatically`. I believe that doing this will not add the upgrade/downgrade different to their period-end charge/invoice. We only assume this because there is no field
on an invoice to refer to the plan change (which it shouldn't). We assume that the system will look for an invoice of plan change at period-end, and if it finds one, it won't add
it to the charge items.

## Developer Notes

Stripe *has* a .NET Core client, which may be Nuget-Installed, so integration with a .NET Core app will be seamless and easy.

