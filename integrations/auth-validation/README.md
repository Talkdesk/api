## Implementing an Authorization Validator

When a user activates an integration for his Talkdesk account, a POST will be made to the configured endpoint with the user's
information for the configured authorization fields. The bridge should validate the credentials with the external system in order to the integration to be properly configured for the account.

### Request

```json
{
    "auth": {
        <auth_field>: <auth_field_value>,
        <auth_field>: <auth_field_value>,
        ...
    }
}
```

### Steps

1. Process the "auth" to check if they correspond to what was configured in Talkdesk

2. Make a request to the external system to validate the given credentials.

3. Return 204 if the credentials are valid, 401 otherwise.
