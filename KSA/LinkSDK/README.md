# Lean Link SDK (Web) — KSA `connect`

A static HTML page that loads the Lean Link SDK (KSA environment) and launches the **`connect`** flow to link a customer's bank account with Lean.

The SDK is shipped as a UMD bundle and attaches to the JavaScript global `window.LeanV2`.

---

## Running the demo locally

1. Fill in `app_token`, `customer_id`, and `customer_access_token` at the top of the inline `<script>` block.
2. Open `LinkSDK-demo-ksa.html` in a browser.
3. Click **Connect** to launch the SDK against the KSA environment.


## Test users with Lean Mock Bank

The Lean Mock Bank users mirror a realistic bank authentication and consent experience in the Sandbox environment. Use the credentials below to test Link SDK in your application. Go integration tab in [KSA Lean Dashboard](https://dev.sa.leantech.me/):

![lean-postman-configuration](/images/test_users.png)

You can generate transactions for a test users by Clicking the "three dots" icon:

![lean-postman-configuration](/images/test_users_generate_transactions.png)


## How LinkSDK is wired

```html
<!-- 1. The SDK script (KSA production) -->
<script src="https://cdn.leantech.me/link/sdk/web/v2/prod/sa/latest/Lean.min.js"></script>

<!-- 2. The mount point the SDK injects into -->
<div id="lean-link"></div>

<!-- 3. Inline <script> that calls connect() -->
<script>
  const app_token = "<your-app-token>";
  const customer_id = "<customer-id>";
  const customer_access_token = "<customer-access-token>";

  function cb(state) {
    console.log(state);
  }

  function connect() {
    LeanV2.connect({
      app_token: app_token,
      customer_id: customer_id,
      access_token: customer_access_token,
      permissions: ["identity", "balance", "accounts", "transactions"],
      success_redirect_url: "https://www.leantech.me/?success=true",
      fail_redirect_url: "https://www.leantech.me/?success=false",
      sandbox: false,
      show_consent_explanation: true,
      callback: cb
    });
  }
</script>
```

The mount point `<div id="lean-link"></div>` is **required** — the SDK looks up this element by id and renders its UI into it.

---

## CDN URL (KSA)

| Environment | URL |
| ----------- | --- |
| KSA Production | `https://cdn.leantech.me/link/sdk/web/v2/prod/sa/latest/Lean.min.js` |
| KSA Sandbox    | Same bundle — pass `sandbox: true` in the config |

---

## `connect` method

`LeanV2.connect(config)` launches the Connect flow:

1. Customer selects a bank.
2. Customer authenticates with the bank.
3. Customer grants the requested data / payments permissions.

The method validates the config, mounts a React app into `#lean-link`, and renders the flow. Validation errors only surface when `sandbox: true`; in production an invalid config silently aborts mounting.

### Required fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `app_token`   | `string`       | Your Lean application token from the Lean dashboard. |
| `customer_id` | `string`       | Unique identifier of the end customer in your system. |
| `permissions` | `Permission[]` | Permissions to request. Must contain at least one value. See [Permissions](#permissions). |
| `access_token`| `string`       | A valid Lean JWT with customer scope. When provided, the SDK skips its own auth step and uses this token as `Authorization: Bearer` on every API request. Must include an `exp` claim with at least **10 minutes** of remaining validity. |
| `success_redirect_url`         | `string`  | URL to redirect to on successful completion. |
| `fail_redirect_url`            | `string`  | URL to redirect to on failure / cancellation. |
| `show_consent_explanation`     | `boolean` | Show an explanation screen before the customer grants consent. |

### Optional fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `access_from`              | `string`         |(ISO 8601 date) Start of the data access window. |
| `access_to`                | `string`         |(ISO 8601 date) End of the data access window. |
| `sandbox`                  | `boolean`        | Run against the Lean sandbox environment. |
| `language`                 | `"en"` or `"ar"` | Force the SDK UI language. Arabic auto-applies RTL. |
| `customization`            | `Customization` | Theme / appearance overrides. See [Customization](#customization). |
| `callback`                 | `(data: CallbackData) => void` | Invoked on SDK lifecycle events. See [Callback](#callback). |

---

## Permissions

`permissions` is an array of strings from the `Permission` enum.

### Data permissions
- `"accounts"`
- `"identity"`
- `"balance"`
- `"transactions"`
- `"beneficiaries"`
- `"identities"`
- `"scheduled_payments"`
- `"direct_debits"`
- `"standing_orders"`

Example:

```js
permissions: ["accounts", "identity", , "transactions", "balance"]
```

---

## Customization

All fields are optional. Colour fields accept HEX (`#RRGGBB`), RGB (`rgb(r,g,b)`), or named CSS colours. The dialog background fields additionally accept CSS `linear-gradient(...)` values.

```ts
{
  theme_color: string;                   // primary brand colour (light mode)
  theme_color_dark: string;              // primary brand colour (dark mode)
  button_text_color: string;             // light-mode button text
  button_text_color_dark: string;        // dark-mode button text
  link_color: string;                    // text-link colour
  overlay_color: string;                 // modal overlay colour
  button_border_radius: number | string; // px number, numeric string, or "pill"
  dialog_mode: "contained" | "uncontained";
  dialog_background_color_light: string; // colour or linear-gradient(...)
  dialog_background_color_dark: string;  // colour or linear-gradient(...)
  force_appearance: "light" | "dark";    // override system appearance
}
```

Example:

```js
customization: {
  theme_color: "purple",
  button_text_color: "white",
  button_border_radius: "pill",
  force_appearance: "light"
}
```

---

## Callback

`callback` receives a partial `CallbackData` object — fields are only populated when relevant to the event:

```ts
{
  status: "SUCCESS" | "ERROR" | "CANCELLED" | "REDIRECT" | "LINK_CLOSED_PROGRAMMATICALLY";
  message: string;
  bank: { bank_identifier?: string; is_supported: boolean };
  exit_point: string;
  secondary_status: string;
  last_api_response: string;
  user_exit_intent: string;
  exit_intent_point: string;
  exit_survey_reason: string;
  lean_correlation_id: string;
  open_banking_redirect_url: string;
}
```

The `status` values come from the `CloseType` enum:

- `SUCCESS` — flow completed successfully.
- `ERROR` — flow ended in error.
- `CANCELLED` — customer closed the SDK before completion.
- `REDIRECT` — flow handed control to an Open Finance redirect.
- `LINK_CLOSED_PROGRAMMATICALLY` — the SDK was closed by host-page code.

Example:

```js
function cb(state) {
  console.log("Lean SDK event:", state.status, state);
}
```

---

## Full `connect` example 

```js
LeanV2.connect({
  app_token: APP_TOKEN,
  customer_id: CUSTOMER_ID,
  access_token: CUSTOMER_ACCESS_TOKEN,
  permissions: ["identity", "balance", "accounts", "transactions"],
  // access_from: "2024-09-05",
  // access_to:   "2024-12-09",
  success_redirect_url: "https://www.leantech.me/?success=true",
  fail_redirect_url:    "https://www.leantech.me/?success=false",
  sandbox: false,
  show_consent_explanation: true,
  callback: (state) => console.log(state)
});
```

---

## More details 
* [Public Lean Documentation](https://docs.leantech.me/v2.0-KSA/docs/web)
