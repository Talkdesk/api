# Implementing an Action Executor

Talkdesk will execute a configured action on the bridge when a trigger of an automation where that action takes part happens on the system. Actions can also be executed from a request of a user with manually input data. An action execution request POST will send the following parameters.

## Request

### Reference

* `auth`
    * **Type:** Hash
    * **Description:** A hash of authentication fields containing a user's credentials within the external service. The keys correspond to the fields asked by the integration when configuring it with Talkdesk.

* `meta`
    * **Type:** Hash
    * **Description:** A hash of meta fields containing accessory information that is useful for the bridge to fullfil this request.

    * `meta.contact_external_id`
        * **Type:** String, optional.
        * **Description:** For contacts that were previously synchronized through the bridge, if this action is executed in the scope of a contact, Talkdesk sends its id in the external service for direct identification.

* `data`
    * **Type:** Hash
    * **Description:** A hash of data fields and respective values the bridge needs to execute the action in the external service. The keys correspond to the fields asked by the integration through the action configuration.

### Example

```json
{
    "auth": {
        "username": "john.doe@example.com",
        "password": "605b32dd"
    },
    "meta": {
        "contact_external_id": "1",
    }
    "data": {
        "subject": "Call missed in Talkdesk",
        "description": "A call from Jane Doe was missed."
    }
}
```

## Steps

1. Process the "auth" request fields to configure own external system client authorization parameters.

2. Use contextual information passed as a meta field (such as the contact id in the external system) to correctly associate actions to the entities in the external system.

2. Process every data parameters and fulfil a request to execute the action on the external system.

3. Return a 201 HTTP status code if it succeeds, 400 otherwise.