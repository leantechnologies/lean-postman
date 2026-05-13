# Lean Postman Collections

Welcome to the Postman Collections Quickstart Guide! If you're looking for a quick and easy way to get started with the Lean API with no additional code, this is the place for you.


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
