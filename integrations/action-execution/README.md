**This documentation is no longer up to date - please refer to our current documentation at [https://docs.talkdesk.com/docs/integrations](https://docs.talkdesk.com/docs/integrations).**

# Implementing an Action Executor

Talkdesk will execute a configured action on the bridge when a trigger of an automation where that action takes part happens on the system. Actions can also be executed from a request of a user with manually input data. An action execution request POST will send the following parameters.

## Request

### Reference

* `auth`
    * **Type:** Hash
    * **Description:** A hash of authentication fields containing a user's credentials within the external service. The keys correspond to the fields asked by the integration when configuring it with Talkdesk.

* `meta`
    * **Type:** Hash
    * **Description:** A hash of meta fields containing accessory information that is useful for the bridge to fulfil this request.

    * `meta.contact_external_id`
        * **Type:** String, optional.
        * **Description:** For contacts that were previously synchronized through the bridge, if this action is executed in the scope of a contact, Talkdesk sends its id in the external service for direct identification.
    * `meta.agent_external_id`
        * **Type:** String, optional.
        * **Description:** For agents that were previously synchronized through the bridge, this field will appear if the agent that executed the action is present in Talkdesk.
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
        "agent_external_id": "2"
    },
    "data": {
        "subject": "Call missed in Talkdesk",
        "description": "A call from Jane Doe was missed."
    }
}
```

## Steps

1. Use the "auth" fields to authenticate on behalf of the user within the external service. Talkdesk makes sure that all mandatory fields were filled as expected.

2. Use contextual information passed as a meta field (such as the contact id in the external service) to correctly associate actions to the entities in the external service. Also,
if possible assign the given agent to the executed action.

3. Build a request based on the data parameters and send it to the external service.

4. Return a 201 HTTP status code if it succeeds, 400 otherwise.

## Response

### Reference
* `message`
  * **Type:** String, optional
  * **Description:** A generic message describing the execution result. Ex: `Case successfully created`, `Case creation failed, reason:`.
* `external_id`
  * **Type:** String, optional
  * **Description:** For actions that create a resource in the external service (other than contact). `external_id` represents the resource `id`
  in the external service.
* `contact_external_id`
  * **Type:** String, optional
  * **Description:** For actions that create a contact in the external service. `contact_external_id` represents contact `id` in the external service.
* `contact_external_type`
  * **Type:** String, optional
  * **Description:** For actions that create a contact in the external service. `contact_external_type` represents contact `type` of in the external service.
* `contact_external_url`
  * **Type:** String, optional
  * **Description:** For actions that create a contact in the external service. `contact_external_url` represents contact `url` in the external service. **NOTE:** Required for contact pop.
