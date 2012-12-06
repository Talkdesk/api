## Implementing an Interaction Retriever

Talkdesk's interaction updater will send an HTTP POST request to the bridge in order to retrieve a contact's interactions with the external system whenever that contact is being displayed to a user. It passes along the following parameters:

### Request

```json
{
    "auth": {
        <auth_field>: <auth_field_value>,
        <auth_field>: <auth_field_value>,
        ...
    },
    "meta": {
        "interaction_types": [
            <interaction_type>,
            <interaction_type>,
            ...
        ]
    },
    "data": {
        "contact_external_id": <contact_external_id>,
        "contact_name": <contact_name>,
        "contact_emails": [
            <contact_email>,
            …
        ],
        "contact_phones": [
            <contact_phone>,
            …
        ]
    }
}
```

### Steps

1. Process the "auth" request fields to configure your own external system client authorization parameters.

2. Select the type of interactions to retrieve by looking at the `interaction_types` "meta" field.

3. Make one or more requests to the external system to retrieve interactions from contacts identified by one or more of the "data" fields. These fields provide contact-related information that the bridge can use to identify the contact within the external system.

4. Adapt these interactions to Talkdesk's format, by returning a JSON array of the following structure:

### Response

```json
[
    {
        'id':            <contact id in the external system>
        'subject':       <interaction subject or title>
        'description':   <a discription/preview/body of the interaction>
        'date':          <interaction date>
        'url':           <interaction URL in the external system>
        'type':          <interaction type matching one of the asked types in the "meta" field of the request>
        'priority':      <one of none/low/normal/high/urgent>
        'status':        <one of new/open/pending/solved/closed>
    },
    ...
]
```