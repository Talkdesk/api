## Implementing an Action Executor

Talkdesk will execute a configured action on the bridge when a trigger of an automation where that action takes part happens on the system. Actions can also be executed from a request of a user with manually input data. An action execution request POST will send the following parameters.

### Request

```json
{
    "auth": {
        <auth_field>: <auth_field_value>,
        <auth_field>: <auth_field_value>,
        ...
    },
    "meta": {
        "contact_external_id": <contact_external_id>,
    }
    "data": {
        <action_data_field>: <action_data_value>,
        <action_data_field>: <action_data_value>,
        <action_data_field>: <action_data_value>,
    }
}
```

### Steps

1. Process the "auth" request fields to configure own external system client authorization parameters.

2. Use contextual information passed as a meta field (such as the contact id in the external system) to correctly associate actions to the entities in the external system.

2. Process every data parameters and fulfil a request to execute the action on the external system.

3. Return a 201 HTTP status code if it succeeds, 400 otherwise.