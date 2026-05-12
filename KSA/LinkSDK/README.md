# Lean Link SDK — Demo Pages

A small set of static HTML pages that load the Lean Link SDK and expose each
SDK method as a button. Use them to **manually test the SDK** in a real browser
without writing any integration code.

There is one page per **environment** (`AE01`, `SA01`) and per **SDK major
version** (V1 and V2). The only differences between pages are:

- The JavaScript global they attach (`window.Lean` for V1, `window.LeanV2` for V2).
- Which buttons / methods are exposed (V2 adds a few V2-only methods).

The HTML, CSS, and button layout are otherwise identical.

---

## 1. Files at a glance

| File                  | Region | Env       | SDK | CDN bundle                                                          | V2-only buttons                                       |
| --------------------- | ------ | --------- | --- | ------------------------------------------------------------------- | ----------------------------------------------------- |
| `ae01-index.html`     | UAE    | Prod      | V1  | `cdn.leantech.me/link/sdk/web/latest/Lean.min.js`                   | —                                                     |
| `ae01-v2-index.html`  | UAE    | Prod      | V2  | `cdn.leantech.me/link/sdk/web/v2/prod/ae/latest/Lean.min.js`        | Capture Redirect, Manage Consents, Auth VOD          |
| `sa01-index.html`     | KSA    | Prod      | V1  | `cdn.leantech.me/sa/link/sdk/web/latest/Lean.min.js`                | (no V2 page exists for KSA)                           |

**Pick a file by the region whose SDK you want to test**, then choose V1
or V2 based on what your integration targets. The buttons (Link, Connect,
Reconnect, Pay, …) behave the same across files — only the SDK build behind
them differs.

---

## 2. Running the pages

These are plain static HTML files. Three options:

### Open directly (fastest)

```bash
open ae01-v2-index.html        # macOS
xdg-open ae01-v2-index.html    # Linux
start ae01-v2-index.html       # Windows
```


---

## 3. How each page is wired

Every page has the same shape:

```html
<!-- 1. The SDK script -->
<script src="https://cdn.leantech.me/.../Lean.min.js"></script>

<!-- 2. The mount point the SDK injects into -->
<div id="lean-link"></div>

<!-- 3. Inline <script> defining one function per button -->
<script>
  function connect()    { Lean.connect({ ... }) }
  function reconnect()  { Lean.reconnect({ ... }) }
  function pay()        { Lean.pay({ ... }) }
  /* etc. */
</script>
```

Clicking a button calls the matching `Lean.<method>(...)` (V1) or
`LeanV2.<method>(...)` (V2) call with whatever the function body contains.

> **V1 vs V2 global:** Pages that load a V1 bundle expose `window.Lean`. Pages
> that load a V2 bundle expose `window.LeanV2`. If you call the wrong one from
> the DevTools console you will get `Lean is not defined` or
> `LeanV2 is not defined`.

---

## 4. Filling in the empty values

Most pages contain function bodies like this:

```js
function pay() {
  Lean.pay({
    app_token,
    payment_intent_id: "0c62deb3-5737-4268-8bd4-d55d5d56e5b1",
    end_user_id,
    sandbox: true,
    callback: cb,
  })
}
```

The bare identifiers (`app_token`, `end_user_id`, `customer_id`,
`payment_destination_id`, `access_token`, `consent_id`, `permissions`,
`customer_name`) are **intentionally undefined** in the file. You have to
provide them yourself. There are two equally valid ways:

### Way 1 — Define them in DevTools before clicking

Open DevTools (`F12` / `Cmd+Opt+I`) → Console, paste:

```js
var app_token            = "YOUR_APP_TOKEN"
var customer_id          = "YOUR_CUSTOMER_ID"
var end_user_id          = "YOUR_END_USER_ID"
var payment_destination_id = "YOUR_PAYMENT_DESTINATION_ID"
var access_token         = "YOUR_JWT"
var consent_id           = "YOUR_CONSENT_ID"
var customer_name        = "Your Customer Name"
var permissions          = ["identity", "accounts", "balance", "transactions"]
```

Then click the button. Define only the variables the button you're about to
click needs (see the reference in section 5).

### Way 2 — Hardcode them in the file

Edit the file and replace each bare identifier with a string literal:

```js
function pay() {
  LeanV2.pay({
    app_token: "abcd-1234-…",
    payment_intent_id: "0c62deb3-…",
    end_user_id: "f05b7212-…",
    sandbox: true,
    callback: cb,
  })
}
```

Pick this way if you'll come back to the same flow repeatedly.

### Where to get each value

| Value                      | Where it comes from                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------ |
| `app_token`                | Lean dashboard → your Application's settings page.                                   |
| `customer_id`              | Returned by `POST /customers` on Lean's API. One per end-customer.                   |
| `end_user_id` (V2)         | Returned by `POST /customers/{id}/end-users` on Lean's API.                          |
| `payment_destination_id`   | Returned by `POST /payments/destinations`. Identifies the merchant bank account.     |
| `payment_source_id`        | Returned after a successful `createPaymentSource` flow — represents a saved payer.   |
| `payment_intent_id`        | Returned by `POST /payments/intents`. Created once per payment you want to collect.  |
| `access_token`             | OAuth JWT minted via Lean's auth endpoint.               |
| `consent_id`               | Returned when a customer grants a consent (manageConsents / authorizeConsent flow).  |
| `consent_attempt_id`       | Returned when a redirect-based consent is started; needed for `captureRedirect`.     |
| `reconnect_id`             | Returned with each completed Connect — needed to re-establish that same connection.  |
| `permissions`              | One or more of `identity`, `accounts`, `balance`, `transactions`, `payments`.        |
| `bank_identifier`          | Lean bank code, e.g. `ENBD_OF_UAE`, `ANB_SAU`, `LEANMB1_SAU`. See Lean's bank list.  |
| `customer_name`            | The display name shown to the end-user during POA / verifyAddress.                   |

Refer to Lean'leantech.me for the live list of
banks, permissions, and the API endpoints that mint each of these IDs.

---

## 5. SDK method reference

Every method takes a single options object. The table below lists what each
button calls, the **required** keys, the **optional** keys, and what each key
does. V1 calls go on `Lean.*`; V2 calls go on `LeanV2.*`. Where a method exists
on both, the signature is the same unless noted.

### `link(options)` — V1 only

Lightweight connect for read-only data access (no payments).

| Key                    | Required | Notes                                                          |
| ---------------------- | -------- | -------------------------------------------------------------- |
| `app_token`            | yes      | Your application token.                                        |
| `customer_id`          | yes      | The end-customer you're linking.                               |
| `permissions`          | yes      | Array; e.g. `["identity", "balance", "accounts", "transactions"]`. |
| `sandbox`              | yes      | `true` for Lean sandbox, `false` for live.                     |
| `success_redirect_url` | yes      | Where to redirect after success.                               |
| `fail_redirect_url`    | yes      | Where to redirect after failure.                               |
| `callback`             | no       | `(state) => …` invoked with state updates.                     |

### `connect(options)` — V1 and V2

The main bank-linking flow. Customer picks a bank and authenticates.

| Key                       | Required | Notes                                                                                              |
| ------------------------- | -------- | -------------------------------------------------------------------------------------------------- |
| `app_token`               | yes      |                                                                                                    |
| `customer_id`             | yes      |                                                                                              |
| `permissions`             | yes      | Must include `payments` if the connection will be used for paying.                                 |
| `sandbox`                 | yes      |                                                                                                    |
| `payment_destination_id`  | when paying | Required only if you plan to initiate payments from this connection.                            |
| `bank_identifier`         | no       | Skip the bank picker by passing a specific bank code, e.g. `ENBD_OF_UAE`.                          |
| `account_type`            | no       | `BUSINESS` for SME flows; defaults to retail.                                                      |
| `access_from`             | no       | ISO date — start of the consent window (KSA / OF flows).                                           |
| `access_to`               | no       | ISO date — end of the consent window.                                                              |
| `success_redirect_url`    | yes      |                                                                                                    |
| `fail_redirect_url`       | yes      |                                                                                                    |
| `end_user_id`             | V2 only  | Required on V2 if your tenant uses end-user scoping.                                               |
| `access_token`            | V2 only  | OAuth JWT. Required if your app uses OAuth instead of `app_token`-only auth.                       |
| `show_consent_explanation`| no       | KSA-specific. Shows the explanatory consent screen.                                                |
| `callback`                | no       |                                                                                                    |

### `reconnect(options)` — V1 and V2

Re-establish a previously created connection without going through the full
icker.

| Key            | Required | Notes                                                |
| -------------- | -------- | ---------------------------------------------------- |
| `app_token`    | yes      |                                                      |
| `reconnect_id` | yes      | Returned with each completed `connect` event.        |
| `sandbox`      | yes      |                                                      |
| `show_logs`    | no       | Verbose logs to the browser console.                 |
| `callback`     | no       |                                                      |

### `createPaymentSource(options)` — V1 and V2

Save a customer's bank as a reusable payment source.

| Key                      | Required | Notes |
| ------------------------ | -------- | ----- |
| `app_token`              | yes      |       |
| `customer_id`            | yes      |       |
| `payment_destination_id` | yes      |       |
| `sandbox`                | yes      |       |
| `show_logs`              | no       |       |
| `callback`               | no       |       |

### `updatePaymentSource(options)` — V1 and V2

Refresh an existing payment source (e.g. to re-authorise expired access).

| Key                      | Required | Notes                              |
| ------------------------ | -------- | ---------------------------------- |
| `app_token`              | yes      |                                    |
| `customer_id`            | yes      |                                    |
| `payment_source_id`      | yes      | The source you want to update.     |
| `payment_destination_id` | yes      |                                    |
| `sandbox`                | yes      |                                    |
| `end_user_id`            | V2 only  |                                    |
| `callback`               | no       |                                    |

### `pay(options)` — V1 and V2

Trigger a payment against an already-created payment intent.

| Key                    | Required s                                                              |
| ---------------------- | -------- | ------------------------------------------------------------------ |
| `app_token`            | yes      |                                                                    |
| `payment_intent_id`    | yes      | One intent per payment; create via Lean API beforehand.            |
| `sandbox`              | yes      |                                                                    |
| `end_user_id`          | V2 only  |                                                                    |
| `access_token`         | V2 only  | OAuth JWT, if your app uses OAuth.                                 |
| `success_redirect_url` | yes      |                                                                    |
| `fail_redirect_url`    | yes      |                                                                    |
| `callback`             | no       |                                                                    |

### `authorize(options)` — V1 and V2

Authorise one or more existing payment intents (used in flows where payment
creation and authorisation are split).

| Key                  | Required | Notes                                                                       |
| -------------------- | -------- | --------------------------------------------------------------------------- |
| `app_token`          | yes      |                                                                             |
| `customer_id`        | yes      |                                                                             |
| `payment_intent_ids` | yes      | **Array** of intent IDs — note the plural and the array type.               |
| `end_user_id`        | yes (V2) |                                                                             |
| `sandbox`            | yes      |                                                                             |

> Note: the button is labelled **"Authorise"** but the method is spelled
> `authorize` (US).

### `createBeneficiary(options)` — V1 and V2

Add a beneficiary that can later receive payments.

| Key                      | Required | Notes |
| ------------------------ | -------- | ----- |
| `app_token`              | yes      |       |
| `customer_id`            | yes      |       |
| `payment_source_id`      | yes      |       |
| `payment_destination_id` | yes      |       |
| `sandbox`                | yes      |       |
| `end_user_id`            | V2 only  |       |
| `callback`               | no       |       |

### `captureRedirect(options)` — V2 only (`ae01-v2-index.html`)

Completes a redirect-based consent flow once the bank has redirected back.

| Key                  | Required | Notes                                                            |
| -------------------- | -------- | ---------------------------------------------------------------- |
| `app_token`          | yes      |                                                                  |
| `customer_id`        | yes      |                                                                  |
| `consent_attempt_id` | yes      | Returned when you started the consent.                           |
| `sandbox`            | yes      |                                                                  |
| `access_token`       | no       |                                                                  |

### `manageConsents(options)` — V2 only (`ae01-v2-index.html`)

Opens the consent-management UI for an existing consent.

| Key            | Required | Notes |
| -------------- | -------- | ----- |
| `app_token`    | yes      |       |
| `customer_id`  | yes      |       |
| `consent_id`   | yes      |       |
| `access_token` | yes      | OAuth JWT — required to read consents. |
| `sandbox`      | yes      |       |
| `callback`     | no       |       |

### `authorizeConsent(options)` — V2 only ("Auth VOD" in `ae01-v2-index.html`)

Authorise an existing consent (VOD = Verification of Data).

| Key                    | Required | Notes                                                       |
| ---------------------- | -------- | ----------------------------------------------------------- |
| `app_token`            | yes      |                                                             |
| `customer_id`          | yes      |                                                             |
| `consent_id`           | yes      |                                                             |
| `sandbox`              | yes      |                                                             |
| `success_redirect_url` | yes      |                                                             |
| `fail_redirect_url`    | yes      |                                                             |
| `bank_identifier`      | no       | Optional pre-selection of the bank.                         |
| `access_token`         | no       |                                                             |

---

## 6. The `cb` callback

Every page defines:

```js
function cb(state) {
  console.log(state)
}
```

Several methods accept a `callback:` key. The SDK calls it with a state object
that describes what happened (`{ status: "SUCCESS", message, … }`,
`{ status: "EXIT", … }`, etc.). To inspect:

1. Open DevTools → Console.
2. Click a button whose function passes `callback: cb`.
3. Watch the logged state objects as the flow progresses.

Replace `cb` with your own handler if you want richer behaviour (redirects,
saving IDs, etc.).

---

## 7. Common gotchas

1. **`ReferenceError: app_token is not defined`.**
   You clicked a button before defining its bare identifiers. See section 4.

2. **`Lean is not defined` or `LeanV2 is not defined` in DevTools.**
   You're using the wrong global. V1 pages expose `Lean`; V2 pages expose
   `LeanV2`. Check the `<script src=...>` tag — paths containing `/v2/` are V2.

3. **`401 Unauthorized` from a method that takes `access_token`.**
   Some V2 pages ship with a real JWT hardcoded in their `connect()` body.
   JWTs expire (usually within an hour of issue). Mint a fresh one via Lean's
   auth endpoint and either edit the file or set `var access_token = "…"` in
   DevTools.

4. **The bank picker is empty.**
   Either the SDK couldn't reach the CDN (check the Network tab for the
   `Lean.min.js` request) or the `permissions` you passed aren't supported in
   that environment for any bank. Try the standard set
   `["identity", "accounts", "balance", "transactions"]`.

5. **Different files have different hardcoded UUIDs for the same parameter.**
   The UUIDs baked into these pages are from various test fixtures and are not
   guaranteed to be valid in your tenant. Treat any hardcoded ID as a hint of
   shape (it's a UUID) rather than a working value — replace it with one your
   own account owns.

6. **`sandbox: true` vs `sandbox: false` mismatch inside one file.**
   Several files have one function set to sandbox and another set to live.
   If "Connect works but Pay doesn't", check both `sandbox:` values are
   pointing at the same environment.


---

## 8. Customising a page

To turn one of these into a starter for your own integration:

1. **Pick the right base file** for the region + SDK version you target.
2. **Replace the bare identifiers** with your own `app_token` / `customer_id` /
   etc. (section 4, way 2).
3. **Set `sandbox`** consistently across every function — `true` while
   developing, `false` once you're live.
4. **Replace `cb`** with a callback that does something useful (store the
   returned `reconnect_id`, navigate the user, etc.).
5. **Wire `success_redirect_url` / `fail_redirect_url`** to pages your app
   actually serves in your admin portal.
6. **Trim buttons you don't need** — every `<p class="lead">…</p>` block is
   independent of the others.
