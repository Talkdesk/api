# Implementing a Contact Retriever

Talkdesk will send an HTTP POST to the bridge's configured endpoint when a contact synchronization is run. This request can be slightly different depending on the situation for which it is run:

* __Initial Sync:__ The first time the contact synchronization runs for a given account, no `synchronization_checkpoint` is sent as a "meta" field, meaning that this is a full retrieval of every contact in the external system.

* __Incremental Sync:__ When a contact sync has been made, the bridge can set a `synchronization_checkpoint` in the response so that the next time the bridge is asked to sync contacts it will only retrieve the ones that changed after the checkpoint. Talkdesk will use the `synchornization_checkpoint` "meta" field for that purpose.

* __Next Page:__ For the previous cases, when adapting third-party services that return contacts in pages, the bridge can signal Talkdesk that the returned contacts are partial by setting `next_offset` in the response. If this is the case, Talkdesk will make a new request for every next page by setting the `offset` "meta" field, until all contacts are retrieved, thus no `next_offset` being returned.

## Request

### Reference

* `auth`
    * **Type:** Hash
    * **Description:** A hash of authentication fields containing a user's credentials within the external service. The keys correspond to the fields asked by the integration when configuring it with Talkdesk.

* `meta`
    * **Type:** Hash
    * **Description:** A hash of meta fields containing accessory information that is useful for the bridge to fullfil this request.

    * `meta.synchronization_checkpoint`
        * **Type:** String, optional.
        * **Description:** A generic starting point for the next synchronization with the 3rd party service. For initial synchronizations, this value will not be sent — which effectively means that the bridge should return all the contacts.

    * `meta.offset`
        * **Type:** String, optional.
        * **Description:** A generic offset the bridge should apply when retrieving paginated contacts. Can either return page numbers, absolute numerical offsets or some other form of result iteration. This parameter is omitted in the first call and is only sent if the bridge instructs the synchronizer that there are more records available.

### Example

```json
{
    "auth": {
        "username": "john.doe@example.com",
        "password": "605b32dd"
    },
    "meta": {
        "offset": "2"
        "synchronization_checkpoint": "2012-11-20 23:01:00 UTC",
    }
}
```

## Steps

1. Process the "auth" request fields to configure own external system client authorization parameters.

2. Process `synchronization_checkpoint` and `offset` "meta" fields to prepare request to be made to the external system:

    * If no `synchornization_checkpoint` is sent, do a full contact retrieval.
    * If no `offset` is sent, retrieve the first page of contacts.

3. Make a request to the external system and convert the result to Talkdesk's contact format, as displayed in the response below.

4. Return contacts in ascending order, sorted by whatever field you deem as useful to use as a checkpoint.

5. Set the `synchronization_checkpoint` to the value of the last contact's field you chose as a checkpoint. Talkdesk's contact synchronization system is completely agnostic to the meaning of this field; bridges are responsible for selecting the best possible fit and interpreting it.

6. Set the `next_offset` field to the value of the next page of results when there are more contacts to retrieve. This will instruct Talkdesk's synchronizer that it needs to make another request to the bridge to retrieve the remaining contacts. As with `synchronization_checkpoint`, the system is agnostic to the meaning of this field; bridges can either return page numbers, absolute numerical offsets or some other form of result iteration.

## Response

### Reference

* `next_offset`
    **Type:** String, optional.
    **Description:** Value for next page offset, if there are more records that match the query.

* `synchronization_checkpoint`
    **Type:** String, optional.
    **Description:** Synchronization checkpoint for this batch of results, resulting in an incremental request next time Talkdesk contact synchronization runs. If not set, all contacts from the external service will be retrieved everytime.

* `contacts`
    **Type:** Array
    **Description:** An array of hashes containing contacts ascendingly by the field the bridge chose to mark as the `synchornization_checkpoint`

    * `contacts.<element>.id`
        **Type:** String, unique, mandatory
        **Description:** Contact's id in the external service

    * `contacts.<element>.name`
        **Type:** String, mandatory
        **Description:** Contact's full name

    * `contacts.<element>.company`
        **Type:** String, optional
        **Description:**

    * `contacts.<element>.title`
        **Type:** String, optional
        **Description:** Contact's title within his company

    * `contacts.<element>.emails`
        **Type:** String, optional
        **Description:** An array of Strings containing contact's emails

    * `contacts.<element>.phones`
        **Type:** String, optional
        **Description:** An array of Strings containing contact's phone numbers

    * `contacts.<element>.photo_url`
        **Type:** String, optional
        **Description:** URL location of a contact's photo

    * `contacts.<element>.address`
        **Type:** String, optional
        **Description:** Contact's full address

    * `contacts.<element>.websites`
        **Type:** Array, optional
        **Description:** An array of Strings containing contact's websites

    * `contacts.<element>.twitter`
        **Type:** String, optional
        **Description:** Contact's Twitter username

### Example

```json
{
    "next_offset": "3",
    "synchronization_checkpoint": "2012-12-01 11:21:00 UTC",
    "contacts": [
        {
            "id":          "1",
            "name":        "Jane Doe",
            "title":       "CTO",
            "emails":      ["jane.doe@example.com", "jane@example.com"],
            "phones":      ["+15555555555"],
            "photo_url":   "http://example.com/users/1/photo",
            "company":     "Example Corp.",
            "address":     "37, Famous St., New York, US",
            "websites":    ["www.example.com"],
            "twitter":     "jane_doe"
        },
        {
            "id":          "2",
            "name":        "Joe",
            "title":       "Staff",
            "emails":      ["joe@example.com"],
        }
    ]
}
```
