## Implementing an Agent Retriever

Talkdesk will send an HTTP POST to the bridge's endpoint when an agent synchronization is needed. The request is as follows:

### Request

#### Reference

* `auth`
    * **Type:** Hash
    * **Description:** A hash of authentication fields containing a user's credentials within the external service. The keys correspond to the fields asked by the integration when configuring it with Talkdesk.

### Example

```json
{
    "auth": {
        "username": "john.doe@example.com",
        "password": "605b32dd"
    }
}
```

### Steps

1. Use the "auth" fields to authenticate on behalf of the user within the external service. Talkdesk makes sure that all mandatory fields were filled as expected;

2. Retrieve the agents by requesting to the correct external endpoint;

3. Adapt the data received by returning a JSON in a format that Talkdesk understands.

### Response

#### Reference

* `agents`
    * **Type:** Array, mandatory.
    * **Description:** Each field must contain an agent with the following fields:

* `id`
    * **Type:** String, mandatory.
    * **Description:** The agent id in the external service.

* `name`
    * **Type:** String, mandatory.
    * **Description:** The name of the agent.

* `email`
    * **Type:** String, mandatory.
    * **Description:** The email of the agent.

#### Example

```json
{
    "agents":
    [
        {
            "id": 341231,
            "name": "John Doe",
            "email": "john.doe@unkn.own"
        }
    ]
}
```