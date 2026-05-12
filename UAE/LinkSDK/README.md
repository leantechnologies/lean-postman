# Lean Link SDK — UAE Demo Pages

Six static HTML pages that load the **UAE** Lean Link SDK and expose each SDK
method as a button. Open one in a browser to manually test the SDK — no build,
no install.

## Files

| File                  | Env       | SDK | CDN bundle loaded                                                  |
| --------------------- | --------- | --- | ------------------------------------------------------------------ |
| `ae01-index.html`     | Prod      | V1  | `cdn.leantech.me/link/sdk/web/latest/Lean.min.js`                  |
| `ae01-v2-index.html`  | Prod      | V2  | `cdn.leantech.me/link/sdk/web/v2/prod/ae/latest/Lean.min.js`       |
| `ae02-index.html`     | Staging   | V1  | `cdn.leantech.me/link/sdk/web/staging/ae/latest/Lean.min.js`       |
| `ae02-v2-index.html`  | Staging   | V2  | `cdn.leantech.me/link/sdk/web/v2/staging/ae/latest/Lean.min.js`    |
| `ae03-index.html`     | Dev       | V1  | `cdn.leantech.me/link/sdk/web/dev/ae/latest/Lean.min.js`           |
| `ae03-v2-index.html`  | Dev       | V2  | `cdn.leantech.me/link/sdk/web/v2/dev/ae/latest/Lean.min.js`        |

- **V1** pages expose `window.Lean.*`.
- **V2** pages expose `window.LeanV2.*` and add buttons for V2-only methods
  (`captureRedirect`, `manageConsents`, `authorizeConsent`/Auth VOD,
  `verifyAddress`/POA).

Pick a file by the deployment stage you want to test against (prod / staging /
dev) and by which SDK major version your integration uses.

## Running

```bash
open ae03-v2-index.html        # macOS
xdg-open ae03-v2-index.html    # Linux
start ae03-v2-index.html       # Windows
```

For redirect-based flows (`success_redirect_url` / `fail_redirect_url`) serve
the folder over a tiny HTTP server instead of `file://`:

```bash
npx serve .
# then open http://localhost:3000/ae03-v2-index.html
```

## Filling in the empty values

Most function bodies reference bare identifiers like `app_token`,
`customer_id`, `end_user_id`, `payment_destination_id`, `access_token`,
`consent_id`, `permissions`, `customer_name`. They are **intentionally
undefined** — provide them either:

**A) In DevTools before clicking the button**

```js
var app_token              = "YOUR_APP_TOKEN"
var customer_id            = "YOUR_CUSTOMER_ID"
var end_user_id            = "YOUR_END_USER_ID"
var payment_destination_id = "YOUR_PAYMENT_DESTINATION_ID"
var access_token           = "YOUR_JWT"
var consent_id             = "YOUR_CONSENT_ID"
var customer_name          = "Your Customer Name"
var permissions            = ["identity", "accounts", "balance", "transactions"]
```

**B) By editing the HTML** — replace each bare identifier with a string
literal of your own value.

### Where each value comes from

| Value                      | Source                                                                    |
| -------------------------- | ------------------------------------------------------------------------- |
| `app_token`                | Lean Dashboard → your Application → settings.                             |
| `customer_id`              | `POST /customers` on the Lean API.                                        |
| `end_user_id` (V2)         | `POST /customers/{id}/end-users`.                                         |
| `payment_destination_id`   | `POST /payments/destinations`.                                            |
| `payment_source_id`        | Returned after a successful `createPaymentSource` flow.                   |
| `payment_intent_id`        | `POST /payments/intents`.                                                 |
| `access_token`             | OAuth JWT minted via Lean's auth endpoint.                                |
| `consent_id`               | Returned when a customer grants a consent.                                |
| `consent_attempt_id`       | Returned when a redirect consent is started; required by `captureRedirect`. |
| `reconnect_id`             | Returned with each completed Connect.                                     |
| `permissions`              | Any of `identity`, `accounts`, `balance`, `transactions`, `payments`.     |
| `bank_identifier`          | Lean bank code, e.g. `ENBD_OF_UAE`, `DIB_OF_UAE`, `FAB_OF_UAE`.           |
| `customer_name`            | Display name shown to the user during POA / `verifyAddress`.              |

Full SDK docs and the live list of banks and permissions:
<https://docs.leantech.me>.

## Buttons → SDK methods

Every button calls the matching SDK method with the options inside its `<script>`
function. Required keys per method:

- **Link** — `Lean.link({ app_token, customer_id, permissions, sandbox })` *(V1)*
- **Connect** — `connect({ app_token, customer_id, permissions, sandbox, … })` — add `payment_destination_id` if you'll initiate payments, `bank_identifier` to skip the picker, `access_token` if your app uses OAuth (V2).
- **Reconnect** — `reconnect({ app_token, reconnect_id, sandbox })`
- **Create payment source** — `createPaymentSource({ app_token, customer_id, payment_destination_id, sandbox })`
- **Update Payment Source** — `updatePaymentSource({ app_token, customer_id, payment_source_id, payment_destination_id, sandbox })` *(V2 adds `end_user_id`)*
- **Initiate payment (Pay)** — `pay({ app_token, payment_intent_id, sandbox })` *(V2 adds `end_user_id`, `access_token`)*
- **Authorise payment** — `authorize({ app_token, customer_id, payment_intent_ids: [...], end_user_id, sandbox })` — note the **array**.
- **Create Beneficiary** — `createBeneficiary({ app_token, customer_id, payment_source_id, payment_destination_id, sandbox })`
- **Capture Redirect** *(V2, ae01-v2)* — `captureRedirect({ app_token, customer_id, consent_attempt_id, sandbox })`
- **Manage Consents** *(V2, ae01-v2)* — `manageConsents({ app_token, customer_id, consent_id, access_token, sandbox })`
- **Auth VOD** *(V2, ae01-v2)* — `authorizeConsent({ app_token, customer_id, consent_id, sandbox, success_redirect_url, fail_redirect_url })`
- **POA** *(V2, ae02-v2)* — `verifyAddress({ app_token, customer_id, customer_name, permissions, sandbox })`

Set `sandbox: true` while developing, `false` for live.

## The `cb` callback

Every page defines `function cb(state) { console.log(state) }`. Methods that
take a `callback:` key call it with state updates (`SUCCESS`, `EXIT`, …).
Watch DevTools → Console, or replace `cb` with your own handler.

## Gotchas

1. **`ReferenceError: app_token is not defined`** — you clicked a button
   before defining its bare identifiers. See the section above.
2. **`Lean is not defined` / `LeanV2 is not defined`** — wrong global. V1
   pages expose `Lean`; V2 pages (paths containing `/v2/`) expose `LeanV2`.
3. **`401 Unauthorized` on V2 Connect/Pay** — some V2 pages ship with a
   hardcoded JWT in `access_token`. JWTs expire — mint a fresh one.
4. **Redirect comes back to a blank page** — `file://` can't be the target
   of an HTTPS redirect. Use the HTTP server option above.
5. **Hardcoded UUIDs are placeholders** — the IDs baked into these files are
   shape examples, not values that work in your tenant. Replace them.
6. **`sandbox` flag mismatch inside one file** — different buttons sometimes
   set `true` and `false`; double-check both before debugging "Connect works
   but Pay doesn't".
