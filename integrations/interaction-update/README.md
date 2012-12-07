# Implementing an Interaction Retriever

Talkdesk's interaction updater will send an HTTP POST request to the bridge in order to retrieve a contact's interactions with the external service whenever that contact is being displayed to a user. It passes along the following parameters:

## Request

### Reference

* `auth`
    * **Type:** Hash
    * **Description:** A hash of authentication fields containing a user's credentials within the external service. The keys correspond to the fields asked by the integration when configuring it with Talkdesk.

* `meta`
    * **Type:** Hash
    * **Description:** A hash of meta fields containing accessory information that is useful for the bridge to fullfil this request.

    * `meta.interaction_types`
        * **Type:** Array
        * **Description:** An array of Strings that identify the types of interactions the Talkdesk user is interested in.

* `data`
    * **Type:** Hash
    * **Description:** A hash of data fields identifying the contact for which the interaction retrieval request is made. The bridge should use one or more of these to identify the contact in the external service.

    * `data.contact_external_id`
        * **Type:** String
        * **Description:** For contacts that were previously synchronized through the bridge, Talkdesk sends their id in the external service for direct identification.

    * `data.contact_name`
        * **Type:** String
        * **Description:** The contact's name

    * `data.contact_emails`
        * **Type:** Array
        * **Description:** An array of emails that can be used to search for the user in the external service.

    * `data.contact_phones`
        * **Type:** Array
        * **Description:** An array of phone numbers that can be used to search for the user in the external service.

### Example

```json
{
    "auth": {
        "username": "john.doe@example.com",
        "password": "605b32dd"
    },
    "meta": {
        "interaction_types": ["case", "note"]
    },
    "data": {
        "contact_external_id": "1",
        "contact_name": "Jane",
        "contact_emails": ["jane.doe@example.com", "jane@example.com"],
        "contact_phones": ["+15555555555"]
    }
}
```

## Steps

1. Use the "auth" fields to authenticate on behalf of the user within the external service. Talkdesk makes sure that all mandatory fields were filled as expected.

2. Select the type of interactions to retrieve by looking at the `interaction_types` "meta" field.

3. Make one or more requests to the external service to retrieve interactions from contacts identified by one or more of the "data" fields. These fields provide contact-related information that the bridge can use to identify the contact within the external service.

4. Adapt these interactions to Talkdesk's format, by returning a JSON array of the following structure:

## Response

### Reference

* `id`
    * **Type:** String, unique, mandatory.
    * **Description:** The interaction id in the external service.

* `subject`
    * **Type:** String, mandatory.
    * **Description:** The interaction subject or title.

* `description`
    * **Type:** String, optional.
    * **Description:** A preview of the interaction. Can be a text description or a preview of an email.

* `date`
    * **Type:** String, mandatory.
    * **Description:** The date this interaction occurred or its due date if it is a future interaction (like a task or event).

* `url`
    * **Type:** String, optional.
    * **Description:** URL location of this interaction in the external service.

* `type`
    * **Type:** String, mandatory.
    * **Description:** The type of the interaction. Should match one of the asked types in the request

* `priority`
    * **Type:** String, `none`, `low`, `normal`, `high` or `urgent`
    * **Description:** The interaction's level of urgency.

* `status`
    * **Type:** String, `new`, `open`, `pending`, `solved` or `closed`
    * **Description:** The current state of the interaction.

### Example

```json
[
    {
        "id":            "23",
        "subject":       "A very important matter",
        "description":   "Your order is ready to be shipped",
        "date":          "2012-11-21 17:53:51 UTC",
        "url":           "http://example.com/interactions/23",
        "type":          "case",
        "priority":      "high",
        "status":        "pending"
    }
]
```