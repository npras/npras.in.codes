---
layout: post
title: Recurring Billing Design - notes
excerpt: Incomplete notes taken while reading a book
---

Notes taken while reading [Multitenancy with Rails](https://leanpub.com/multi-tenancy-rails-2). This (incompletely) captures the process of implementing a recurring billing functionality in a web app.

## Formulating Plans
* Create Plans in the Payment Processor's site. Eg: In Braintree sandbox env.
* Store the plans locally in a `Plan` model - name:string, price:float, braintree_id:string
* Have a rake task `import_plans` that is run everytime a Plan is created/updated at the Braintree sandbox side. It will fetch the plans from BT and sync it up locally as rails `Plan` objects

## Creating Subscriptions
### Create a Braintree Customer 
* for every account object created in rails, create a BT Customer (assuming each `Account` will be the entity that subscribes and pays for a plan. It can also be a `User` depending on your usecase)
* Associate the local `account` object with the `braintree_customer_id`

### Choosing a Plan
* Once the rails `account` object, and braintree `customer` object are created, these both needs to be associated with the `plan` that the user is going to choose.
It can be done by adding a `plan_id` to the `Account` model.
* So the `account` object has both `braintree_customer_id` and `plan_id`, both enough to create a subscription.

### Adding a Credit Card Form
* In the same form where they get to choose a plan, ask them for their payment info in another form. This form will come from Braintree Javascript SDK (drop-in UI).
* This form will submit to our app's server (maybe to `accounts/plans#chosen` action), but before that the payment details will go to braintree server directly and will get a `payment_method_nonce` back. This and the user selection of the 'plan' will then come to our server which we need to process.

### Taking User's money
* With this nonce, we can then charge money!
* But before taking user's money, we'll need to first create a Subscription for this user's account at BT server!
* Creating an active subscription automatically charges i guess?

## Changing Subscriptions
Users should be able to change subscriptions too.
So, apart from the "Pick a Plan" page, there should also be a "Switch Plans" page. It should only be accessed to a user who's signed in, and already has a subscription.
Switching should also be restricted based on plans. Disable switching to plans based on usage limits.

## Process Webhooks
* Write code to process webhooks for almost all events that your provider sends. Most important:
  * Subscription Charged Successfully
  * Subscription Charged Unsuccessfully
  * Subscription Canceled
  * Subscription Went Past Due
Take different actions based on these events. For eg: if a subscription is cancelled, send an email from the founder, "What we did wrong? How can we better improve?"
Or if a subscription went past due, block access to the account for that user, and notify him. Ask him to try paying again or change card details.



## Things to be careful about
* Report to the users that occur at BT server side. Eg: invalid credit card
* Errors can happen when creating a customer, when creating a client token, when creating a subscription etc, when user inputs invalid data etc
* Ensure access control to all the important parts of the site. A user shouldn't be able to access a restricted page with just an account, and without selecting and subscribing to a plan.
* You are legally obligated to provide a way to cancel a subscription
* A user who has cancelled his subscription shouldn't be able to access restricted content
* Save locally the current state of an account's BT subscription info (:braintree_subscription_status in an `account` model). This can be used for access control.
* Cancelled accounts shouldn't be accessible to owners as well as the account's users
* Actions like cancelling a subscription, or resubscribing should only be limited to account owners
* Have a 'billing' section in your app where the signed in user should be able to update his billing details.. this page/form can be used when the user has a subscription that's past due.
