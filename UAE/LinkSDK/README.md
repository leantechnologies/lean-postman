# Lean Link SDK — Demo Pages

A  static HTML page that loads the Lean Link SDK and expose each
SDK method as a button. Use them to **manually test the SDK** in a real browser
without writing any integration code.

- The JavaScript global they attach `window.LeanV2`.



---

## How LinkSDK  is wired


```html
<!-- 1. The SDK script -->
<script src="https://cdn.leantech.me/.../Lean.min.js"></script>

<!-- 2. The mount point the SDK injects into -->
<div id="lean-link"></div>

<!-- 3. Inline <script> defining one function per button -->
<script>
  function connect()    { Lean.connect({ ... }) }
  /* etc. */
</script>
```