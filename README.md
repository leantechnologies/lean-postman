# Lean Postman Collections

The fastest way to get started with Lean APIs without writing any code. The repo provides two Postman collections, one per country:

- `KSA/LeanAPI-KSA.postman_collection.json`
- `UAE/Lean-API-UAE.postman_collection.json`

### Importing a collection into Postman

1. In the Postman desktop or web app, click **Import** (top-left, next to the workspace name).
2. On the **Files** tab, pick the JSON file from your local clone of this repo. Or paste a raw GitHub URL on the **Link** tab.
3. Click **Import**. The collection lands in the current workspace.

Set the collection variables (below) before you send any requests, otherwise the auth step will fail.

### Configuration

The collections use [Postman Collection variables](https://learning.postman.com/docs/sending-requests/variables/variables#defining-variables) so credentials only live in one place.

![lean-postman-configuration](/images/collection_variables.png)

1. In the sidebar, open **Collections** and pick the one you want to configure (KSA or UAE).
2. Open the **Variables** tab.
3. Fill in `client_id` and `client_secret` from the **Integration** tab of your Lean Application Dashboard:
   - [KSA Lean Dashboard](https://dev.sa.leantech.me/)
   - [UAE Lean Dashboard](https://dev.leantech.me/)
4. Save.

`client_id` is your application ID and stays visible on the dashboard. `client_secret` is shown once when you generate it; regenerate it and the previous value stops working, so keep it somewhere safe.

![lean-postman-configuration](/images/lean_credentials.png)

---

## LinkSDK playgrounds

Each country folder includes an HTML page that loads the Lean LinkSDK from the CDN and turns each SDK method into a button. Open the file in a browser and you can run the SDK against sandbox or production with no build step.

| Country | Demo file | Details |
| ------- | --------- | ------- |
| KSA     | `KSA/LinkSDK/LinkSDK-demo-ksa.html` | [`KSA/LinkSDK/README.md`](./KSA/LinkSDK/README.md) |
| UAE     | `UAE/LinkSDK/LinkSDK-demo-UAE.html` | [`UAE/LinkSDK/README.md`](./UAE/LinkSDK/README.md) |
