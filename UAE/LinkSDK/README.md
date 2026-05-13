# Lean Link SDK (Web) — UAE `connect`

A static HTML page that loads the Lean Link SDK (UAE environment) and launches the **`connect`** flow to link a customer's bank account with Lean.

The SDK ships as a UMD bundle and attaches to the global `window.LeanV2`.

---

## Running the demo locally

1. Fill in `app_token`, `customer_id`, and `customer_access_token` at the top of the inline `<script>` block.
2. Open `LinkSDK-demo-UAE.html` in a browser (or serve the folder over `http://`).
3. Click **Connect** to launch the SDK against the UAE environment.
4. When configuring the redirection URLs in the connect() flow in the LinkSDK (such as success_redirect_url and failure_redirect_url), please ensure to add the URLs in Lean dashboard under **Settings** tab as follows: 
![lean-postman-configuration](/images/redirect_urls.png)




## How LinkSDK is wired

```html
<!-- 1. The SDK script (UAE production) -->
<script src="https://cdn.leantech.me/link/sdk/web/v2/prod/ae/latest/Lean.min.js"></script>

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

The `<div id="lean-link"></div>` mount point is required. The SDK looks it up by id and renders into it; without it, nothing happens.

---

## CDN URL (UAE)

| Environment | URL |
| ----------- | --- |
| UAE Production | `https://cdn.leantech.me/link/sdk/web/v2/prod/ae/latest/Lean.min.js` |
| UAE Sandbox    | Same bundle. Pass `sandbox: true` in the config. |

To pin a specific SDK version, replace `latest` with a fixed version segment in the URL.

---

## `connect` method

`LeanV2.connect(config)` runs the Connect flow: the customer picks a bank, logs in with it, and grants the data or payments permissions you asked for.

The method validates the config, mounts a React app into `#lean-link`, and renders the flow. Validation errors only surface when `sandbox: true`. In production, an invalid config silently aborts the mount, so test in sandbox first.

### Required fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `app_token`   | `string`       | Your Lean application token from the Lean dashboard. |
| `customer_id` | `string`       | Unique identifier of the end customer in your system. |
| `permissions` | `Permission[]` | Permissions to request. Must contain at least one value. See [Permissions](#permissions). |
| `access_token`| `string`       | A valid Lean JWT with customer scope. When provided, the SDK skips its own auth step and uses this token as the `Authorization: Bearer` header on every API request. The JWT must include an `exp` claim with at least **10 minutes** of remaining validity. |

### Optional fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `end_user_id`              | `string`         | Your internal identifier for the end user. |
| `customer_metadata`        | `string`         | Free-form metadata string attached to the session. |
| `bank_identifier`          | `string`         | Pre-select a bank by its Lean identifier. |
| `account_type`             | `"PERSONAL"` or `"BUSINESS"` | Filter the bank list by account type. |
| `access_from`              | `string`         | Start of the data access window. ISO 8601 date. |
| `access_to`                | `string`         | End of the data access window. ISO 8601 date. |
| `show_consent_explanation` | `boolean`        | Show an explanation screen before the customer grants consent. |
| `success_redirect_url`     | `string`         | URL to redirect to on successful completion (Open Finance flows). |
| `fail_redirect_url`        | `string`         | URL to redirect to on failure or cancellation (Open Finance flows). |
| `destination_alias`        | `string`         | Display name for the payment destination shown in the UI. |
| `destination_avatar`       | `string`         | Logo URL for the payment destination shown in the UI. PNG or JPG. |
| `sandbox`                  | `boolean`        | Run against the Lean sandbox environment. |
| `language`                 | `"en"` or `"ar"` | Force the SDK UI language. Arabic auto-applies RTL. |
| `customization`            | `Customization`  | Theme and appearance overrides. See [Customization](#customization). |
| `callback`                 | `(data: CallbackData) => void` | Called on SDK lifecycle events. See [Callback](#callback). |

---

## Permissions

`permissions` is an array of strings from the `Permission` enum.

### Data permissions

- `"identity"`
- `"balance"`
- `"accounts"`
- `"transactions"`
- `"beneficiaries"`
- `"identities"`
- `"scheduled_payments"`
- `"direct_debits"`
- `"standing_orders"`

### Payments

- `"payments"`: request payment initiation capability.

Example:

```js
permissions: ["identity", "accounts", "transactions", "balance", "payments"]
```

---

## Customization

All fields are optional. Colour fields accept HEX (`#RRGGBB`), RGB (`rgb(r,g,b)`), or a named CSS colour. The dialog-background fields also accept a CSS `linear-gradient(...)` value.

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

`callback` receives a partial `CallbackData` object. Only the fields relevant to the current event are populated:

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

`status` comes from the `CloseType` enum:

- `SUCCESS`: the flow completed.
- `ERROR`: the flow ended in error.
- `CANCELLED`: the customer closed the SDK before finishing.
- `REDIRECT`: control handed off to an Open Finance redirect.
- `LINK_CLOSED_PROGRAMMATICALLY`: host-page code closed the SDK.

Example:

```js
function cb(state) {
  console.log("Lean SDK event:", state.status, state);
}
```

---

## Full `connect` example (UAE)

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
- [Public Lean Documentation](https://docs.leantech.me/docs/web)
