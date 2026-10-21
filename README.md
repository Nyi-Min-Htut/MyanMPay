# MyanMyanPay Sandbox Integration

A Laravel 9 project built for the MyanMyanPay Developer Onboarding assessment.
It connects to the sandbox, creates a real test payment, shows the QR, and
updates the status when a webhook arrives.

I followed the official MyanMyanPay API documentation throughout. All endpoint
names, SDK method calls, and payload formats in this project come from the
docs, not from guessing.

## Setup

Install dependencies, create your `.env`, generate the app key, then fill in
your sandbox credentials: App ID, Publishable Key, Secret Key, and callback
URL. After that, clear the config cache and start the server. The test page
is at `/mmpay-test`.

## How it works

The user opens an order page with a product summary and an amount field.
Clicking Pay validates the amount, then calls the SDK to create a payment.
The SDK handles the handshake and signing. The response contains an EMVCo
QR string, which the app renders as an SVG image. Status starts as pending.

While the QR is visible, the page checks status every few seconds. It looks
at two things: the API status and any webhook that has arrived. If a webhook
is present, that takes priority. Otherwise the API status is shown.

When MyanMyanPay completes a payment, it sends a webhook to the callback URL.
The app verifies the signature using the SDK's built-in method, then stores
the payload and updates the status.

## What is used

Laravel 9, the official MyanMyanPay PHP SDK, and endroid/qr-code for
rendering the QR string as an image. The webhook route lives in `api.php`
because web routes have CSRF protection that would block external requests.

## About the sandbox scanner

The sandbox I was given did not include a physical scanner app, so I could
not scan the QR to trigger a real webhook. Instead of faking a payment, I
added a test button on the page that builds a signed payload in the same
format MyanMyanPay uses and sends it through the real webhook handler. This
exercises the full code path — signature verification, cache write, and UI
update. The page labels it clearly as a test and shows that the API still
reports pending, because no real payment happened.

If a scanner were available, the same handler would process the real webhook
with no code changes. Only the callback URL would need to become publicly
reachable via ngrok.

## What I tested

path: payment is created, the QR comes back from the real sandbox API,
and it renders on the page.

Webhook: a signed request reaches the real handler, signature verification
passes, and the cache is updated.

Failure scenarios: an invalid amount is caught by form validation before
the API is called, and an API failure or invalid key is caught by try-catch
with a clear error message.


## Security

Credentials are only in `.env`, which is gitignored. The `.env.example` file
contains placeholders only. Webhook requests without a valid signature are
rejected. The test webhook button is restricted to the local environment.

## Submission contents

Source code, this README, `.env.example` with placeholders, a screen
recording, and a live walkthrough during the interview.