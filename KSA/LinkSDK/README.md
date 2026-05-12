# Lean Link SDK — KSA Demo Pages

Two static HTML pages that load the **KSA** Lean Link SDK and expose each SDK
method as a button. Open one in a browser to manually test the SDK — no build,
no install.

## Files

| File                  | Env       | SDK | CDN bundle loaded                                              |
| --------------------- | --------- | --- | -------------------------------------------------------------- |
| `sa01-index.html`     | Prod      | V1  | `cdn.leantech.me/sa/link/sdk/web/latest/Lean.min.js`           |
| `sa02-index.html`     | Staging   | V1  | `cdn.leantech.me/link/sdk/web/staging/sa/latest/Lean.min.js`   |

Both files expose `window.Lean.*`. KSA does not currently ship a V2 page
(`LeanV2` is UAE-only at the moment).

## Running

```bash
open sa01-index.html        # macOS
xdg-open sa01-index.html    # Linux
start sa01-index.html       # Windows
```

For redirect-based flows (`success_redirect_url` / `fail_redirect_url`) serve
the folder over a tiny HTTP server instead of `file://`:

```bash
npx serve .
# then open http://localhost:3000/sa01-index.html
```

## Filling in the values

The KSA pages ship with `app_token` and `customer_id` defined at the top of
the `<script>` block, plus per-function values inside each `Lean.<method>(...)`
call. Some calls also reference bare identifiers (e.g. `payment_destination_id`).

You have two options:

**A) Define missing identifiers in DevTools before clicking:**

```js
var payment_destination_id = "YOUR_PAYMENT_DESTINATION_ID"
```

**B) Edit the HTML** and replace any value (including the hardcoded `app_token`
and `customer_id` at the top) with your own.

### Where each value comes from

| Value                      | Source                                                                    |
| -------------------------- | ------------------------------------------------------------------------- |
| `app_token`                | Lean Dashboard → your Application → settings.                             |
| `customer_id`              | `POST /customers` on the Lean API.                                        |
| `payment_destination_id`   | `POST /payments/destinations`.                                            |
| `payment_source_id`        | Returned after a successful `createPaymentSource` flow.                   |
| `payment_intent_id`        | `POST /payments/intents`.                                                 |
| `reconnect_id`             | Returned with each completed Connect.                                     |
| `permissions`              | Any of `identity`, `accounts`, `balance`, `transactions`, `payments`.     |
| `bank_identifier`          | Lean bank code, e.g. `ANB_SAU`, `LEANMB1_SAU`, `RIYAD_SAU`.               |

Full SDK docs and the live list of banks and permissions:
<https://docs.leantech.me>.

## Buttons → SDK methods

| Button                | Method                                                                                              |
| --------------------- | --------------------------------------------------------------------------------------------------- |
| Link                  | `Lean.link({ app_token, customer_id, permissions, sandbox })`                                       |
| Connect               | `Lean.connect({ app_token, customer_id, permissions, sandbox, access_from, access_to, account_type, … })` |
| Reconnect             | `Lean.reconnect({ app_token, reconnect_id, sandbox })`                                              |
| Create payment source | `Lean.createPaymentSource({ app_token, customer_id, payment_destination_id, sandbox })`             |
| Update Payment Source | `Lean.updatePaymentSource({ app_token, customer_id, payment_source_id, payment_destination_id, sandbox })` |
| Initiate payment      | `Lean.pay({ app_token, payment_intent_id, sandbox })`                                               |

KSA Connect commonly uses `access_from` / `access_to` (ISO dates — start and
end of the consent window) and `account_type: 'BUSINESS'` for SME flows.
`bank_identifier` skips the bank picker (e.g. `ANB_SAU`).

Set `sandbox: true` while developing, `false` for live.

## The `cb` callback

Both pages define `function cb(state) { console.log(state) }`. Methods that
take a `callback:` key call it with state updates (`SUCCESS`, `EXIT`, …).
Watch DevTools → Console, or replace `cb` with your own handler.

## Gotchas

1. **`ReferenceError: payment_destination_id is not defined`** — you clicked
   Create / Update Payment Source without defining its ID. See above.
2. **`Lean is not defined`** — the SDK script didn't load. Check the Network
   tab for the `Lean.min.js` request.
3. **Expired `access_from` / `access_to` window** — these are ISO dates. If
   they're in the past, KSA banks will refuse the consent. Update them to a
   current range before clicking Connect.
4. **Redirect comes back to a blank page** — `file://` can't be the target of
   an HTTPS redirect. Use the HTTP server option above.
5. **Hardcoded UUIDs are placeholders** — the IDs baked into these files are
   shape examples, not values that work in your tenant. Replace them.
6. **`sandbox` flag mismatch inside one file** — different buttons sometimes
   set `true` and `false`; double-check before debugging.
