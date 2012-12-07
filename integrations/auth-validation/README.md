# Implementing an Authorization Validator

When a user activates an integration for their Talkdesk account, a POST will be made to the configured endpoint with the user's
information for the configured authorization fields. The bridge should validate the credentials with the external service in order to the integration to be properly configured for the account.

## Request

### Reference

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

## Steps

1. Process the "auth" to check if they correspond to what was configured in Talkdesk

2. Make a request to the external service to validate the given credentials.

3. Return 204 if the credentials are valid, 401 otherwise.
