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

1. The "auth" fields contain user submitted values (and/or resulting from the OAuth 2.0 token exchange) for the fields the integration configured within Talkdesk. Use that information as the user authentication within the external service.

2. Build a request to the external service to validate the given credentials. This can be a dummy request just to make sure the credentials are valid, or a more elaborate validation thorugh an authorization endpoint the external service may provide.

3. Return 204 if the request succeeds, 401 otherwise.
