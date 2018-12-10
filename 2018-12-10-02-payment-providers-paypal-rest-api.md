# Payment Providers - PayPal Rest API

PayPal accounts are quite different than the typical payment provider because it requires redirection to PayPal so that the customer can authorize the purchase before actually
allowing the merchant to charge. We will focus on PayPal's new REST API for managing upgrades, downgrades, and cancellations. We shall assume that we have set up our plans,
including prices and durations, and that the user has initially signed up to one of our plans.

This does *not* pertain to using PayPal as a payment provider for credit cards!

In PayPal REST API, a subscription is called a **billing agreement**.

## Cancellation

There is a way to cancel a billing agreement, which is `POST /v1/payments/billing-agreements/{agreement_id}/cancel`. There are no details as to what happens financially.
It is possible that the current billing agreement remains in effect until its termination date, however, recurrent billing will no longer happen. The billing agreement is still
considered active, though it was marked cancel-requested.

## Upgrading and Downgrading

From what I've found online, I don't think there is an automatic scheme for managing upgrades and downgrades. We would have to cancel the current agreement, figure out what to charge
and/or refund, perform the charge and/or refund, and then sign up the user for the new subscription with an new plan.

## The Bottom Line

PayPal REST API is excellent for one-time purchases and has very limited support for recurrent billing. Yes, it can do recurrent billing, however, the documentation doesn't cover the
common instance when a user wants to cancel. There is no support for upgrading and downgrading.

If we want full recurrent billing support including reference transactions, we will need to use platform-specific SDKs for the Express Checkout, which is referred to as the Merchant
.Net SDK. Unfortunately for Asp.NET, it doesn't support Asp.NET Core, only upto 4.x.y.

