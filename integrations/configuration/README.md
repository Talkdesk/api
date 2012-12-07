# Configuring a Talkdesk Integration

Talkdesk has to know a few things about your new integration: what it is called, what kind of authentication mechanism it needs, where it lives and which feature set it provides.

In the following sections we'll go through these configuration options.

## Authentication Configuration

Depending on the third-party service you will be integrating with Talkdesk, some form of user authentication might be necessay. Talkdesk lets the integration configure which authentication fields it needs from the user. You can then use this information to identify the user and make authorized requests on his behalf to the third-party system.

Three authentication types can be used for an integration:

* `none`: Use this if no authentication information is needed
* `custom`: This type of authentication will ask the user for the authentication fields the bridge requires when he first activates the integration for his account. These will be passed transparently to the bridge as authentication parameters when making requests for that user.
* `oauth2`: If the third-party service you are integrating supports OAuth 2.0 user authentication, Talkdesk offers a simple way to ask the user for access to his account within the third-party service.

> Notes on OAuth 2.0 authentication type

> 1. When using OAuth 2.0 authentication, Talkdesk will request the authorization credentials by redirecting the user to the third-party service and trading a verification code for an access token.
> 2. This token will be passed to the bridge as an authentication parameter letting it make requests to the third-party service on behalf of the user.
> 3. If the third-party service issues a refresh token, Talkdesk will save it to refresh the access token when it expires.
> 4. If a bridge makes a request to a third-party service and it fails due to unauthorized access, returning back a 401 to Talkdesk will make it refresh the access token and repeat the request on the bridge passing the new token.

## Integration Configuration

An integration configuration has to provide a JSON configuration based on the following reference:

* `name`
    * **Type:** String, unique, mandatory.
    * **Description:** The system name of the integration.

* `display_name`
    * **Type:** String, mandatory.
    * **Description:** A user-friendly display name for this integration, will be used when displaying the interface.

* `authentication_type`
    * **Type:** String, `none`, `custom` or `oauth2`.
    * **Description:** The type of authentication to use. If `none` or `oauth2` are set, `authentication_fields` is unnecessary. For `custom`, authentication fields must be set.

* `authentication_fields`
    * **Type:** Array, optional when `authentication_type` is set to `none` or `oauth2`, mandatory for `custom`.
    * **Description:** A hash of authentication fields required for the external integration's authentication process. When requests are made to the external integration's bridge, the value to send for these fields is retrieved from the user account.

    * `authentication_fields.<element>`
        * **Type:** String, unique within the element.
        * **Description:** The name of the field to send.

    * `authentication_fields.<elements>.source`
        * **Type:** String, `input`, `default` or `auth_validation`.
        * **Description:** The source of this field.

    * `authentication_fields.<elements>.display`
        * **Type:** String, mandatory.
        * **Description:** Display friendly name when presenting the input on the interface.

    * `authentication_fields.<elements>.mandatory`
        * **Type:** Boolean, mandatory.
        * **Description:** Talkdesk will only validate mandatory fields.

    * `authentication_fields.<elements>.type`
        * **Type:** String, `input` or `oauth`.
        * **Description:** Instruct Talkdesk on how to obtain the field: `input` for user provided fields, `oauth` for fields provided by the OAuth 2.0 authority.

    * `authentication_fields.<elements>.store`
        * **Type:** Boolean, mandatory.
        * **Description:** Controls if the field is to be stored in the configuration. A use case for this is combining type `oauth` with store `false` for field 'code' when requiring Talkdesk to retrieve an oauth code.

    * `authentication_fields.<elements>.help`
        * **Type:** String, optional.
        * **Description:** Hint to display along with the field, on the interface.

    * `authentication_fields.<elements>.format`
        * **Type:** String, optional.
        * **Description:** Template to gather user input, allows a friendlier display on the interface (e.g. `http://talkdesk.com/{{username}}`)

* `contact_synchronization_url`
    * **Type:** String, optional.
    * **Description:** The URL of the integration bridge responsible for contact synchronization. If nil, contact synchronization features will not be available for the integration.

* `interaction_retrieval_url`
    * **Type:** String, optional.
    * **Description:** The URL of the integration bridge responsible for interaction retrieval. If nil, interaction retrieval features will not be available for the integration.

* `auth_validation_endpoint`
    * **Type:** String, optional.
    * **Description:** The URL of the integration bridge responsible of validating if the given credentials authorize the user.

* `interaction_types`
    * **Type:** Array, optional
    * **Description:** An array of Strings that identify which types of interactions this integration can retrieve. Each account will configure a subset of these types it is interested in.

* `description`
    * **Type:** String, mandatory
    * **Description:** A user-friendly description of the service the integration is bridging to.

* `logo_url`
    * **Type:** String, mandatory
    * **Description:** The URL location of the integration service logo to be displayed when configuring the integration.

* `icon_url`
    * **Type:** String, mandatory
    * **Description:** The URL location of the integration service icon to be displayed for retrieved interactions within a contact.


Example:
```json
{
  "name": "zendesk",
  "display_name": "Zendesk",
  "authentication_type": "custom",
  "authentication_configuration": [
    {
      "account": { "source": "input", "display": "Account", "mandatory": true, "type": "input", "store": true, "format": "https://{{account}}.zendesk.com" }
    },
    {
      "email": { "source": "input", "display": "Agent Email", "mandatory": true, "type": "input", "store": true }
    },
    {
      "token": { "source": "input", "display": "API Token", "mandatory": true, "type": "input", "store": true, "help": "Find under Settings > Channels" }
    }
  ],
  "contact_synchronization_endpoint": "https://td-zendesk.herokuapp.com/contact_sync",
  "interaction_retrieval_endpoint": "https://td-zendesk.herokuapp.com/interaction_update",
  "auth_validation_endpoint": 'https://td-zendesk.herokuapp.com/auth_validation',
  "interaction_types": ["ticket"],
  "description": "Centralize all your customer conversations so nothing gets ignored and everything is searchable   from one place. Easily organize, prioritize and engage with others on support requests.",
  "logo_url": "https://talkdeskapp.s3.amazonaws.com/assets/integrations/zendesk.png",
  "icon_url": "https://talkdeskapp.s3.amazonaws.com/assets/integrations/zendesk_icon.png"
}
```

## Configuring Actions

If an integration provides actions for Talkdesk to call, each of those has to provide a JSON configuration based on the following reference:

* `provider`
    * **Type:** String, mandatory
    * **Description:** The integration that provides this action. This has to match the "name" attribute used to configure the integration.

* `name`
    * **Type:** String, mandatory
    * **Description:** Unique identifier of the action

* `display`
    * **Type:** String, mandatory
    * **Description:** User-friendly name used to identify the action

* `description`
    * **Type:** String, mandatory
    * **Description:** User-friendly description that Talkdesk will show to the user

* `endpoint`
    * **Type:** String, mandatory
    * **Description:** The URL of the integration bridge responsible to execute this action.

* `inputs`
    * **Type:** Array, mandatory
    * **Description:** An array of field definitions that this action can receive from Talkdesk in order to be executed in the third-party system.

    * `inputs.<element>.key`
        * **Type:** String
        * **Description:** The identifier of the field to send.

    * `inputs.<element>.name`
        * **Type:** String
        * **Description:** The name of the field to send.

    * `inputs.<element>.mandatory`
        * **Type:** Boolean
        * **Description:** Talkdesk won't execute the action if no value is given for mandatory fields.

Example:
```json
{
  "provider": "zendesk",
  "name": "create_ticket",
  "display": "create a ticket",
  "description": "Create a new ticket in Zendesk",
  "endpoint": ""
  "inputs": [
    {
      "key": "subject",
      "name": "Ticket subject",
      "mandatory": true
    },
    ...
  ],
}
```
