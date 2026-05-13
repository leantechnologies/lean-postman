# Lean Postman Collections

Welcome to the Postman Collections Quickstart Guide! If you're looking for a quick and easy way to get started with the Lean APIs with no additional code, this is the place for you.

The repository ships two ready-to-use Postman collections, one per region:

- `KSA/LeanAPI-KSA.postman_collection.json`
- `UAE/Lean-API-UAE.postman_collection.json`

### Importing a collection into Postman

You can import either (or both) collections into your Postman workspace.

1. In Postman desktop app or web app, click **Import** (top-left, next to the workspace name).
2. Choose the **Files** tab and select the JSON collection file from your local clone of this repository, or paste a raw GitHub URL to the file under the **Link** tab.
3. Click **Import** — Postman adds the collection to the current workspace.



After importing, follow the **Configuration** section below to set the collection variables before sending requests.

### Configuration

The Lean Postman collection uses [Postman Collection variables](https://learning.postman.com/docs/sending-requests/variables/variables#defining-variables) to simplify making API requests.

![lean-postman-configuration](/images/collection_variables.png)

1. Click on the `Lean API collection` 
1. To edit collection variables, click Collections in the sidebar, select a collection you want to view (KSA or UAE)
2. Select the  `Variables` section
3. Copy in your Lean API keys from your Lean Application Dashboard under integration tab into each field:
    * [KSA Lean Dashboard](https://dev.sa.leantech.me/)
    * [UAE Lean Dashboard](https://dev.leantech.me/)

- `client_id`: Your application ID - this can be retrieved from Lean Application Dashboard account.
- `client_secret`: Your client secret - this can be retrieved once from Lean Application Dashboard. Note: subsequent retrievals will invalidate your existing client secret.
The `client_id` and `client_secret` can be retreived under the `Integration` tab:

![lean-postman-configuration](/images/lean_credentials.png)

4. Save your changes and start making Lean API requests!
