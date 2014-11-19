# Implementing a Contact Retriever

Talkdesk will send an HTTP POST to the bridge's configured endpoint when a contact synchronization is run. This request can be slightly different depending on the situation for which it is running:

* __Initial Sync:__ The first time the contact synchronization runs for a given account the bridge should retrieve every contact in the external system. When responding to the initial sync, the bridge can set a `synchronization_checkpoint` that Talkdesk will use from that moment on to only sync contacts incrementally.

* __Incremental Sync:__ When a contact sync has been made and the bridge has set a `synchronization_checkpoint` in the response, Talkdesk will use that in the request that the bridge can use to only retrieve contacts updated after that checkpoint.

* __Next Page:__ In both cases, when the external service only lets the bridge retrieve contacts in pages, the bridge should respond with the partially retrieved settings and set a `next_offset` in the response with the next page to retrieve. When this is set, Talkdesk will immediately make a new request with this value as the `offset` field within "meta" to retrieve the next page of contacts. This will repeat until the bridge returns no `next_offset` in the response, thus meaning that all contacts have been retrieved.

## Request

### Reference

* `auth`
    * **Type:** Hash
    * **Description:** A hash of authentication fields containing a user's credentials within the external service. The keys correspond to the fields asked by the integration when configuring it with Talkdesk.

* `meta`
    * **Type:** Hash
    * **Description:** A hash of meta fields containing accessory information that is useful for the bridge to fulfil this request.

    * `meta.synchronization_checkpoint`
        * **Type:** String, optional.
        * **Description:** A generic starting point for the next synchronization with the 3rd party service. For initial synchronizations, this value will not be sent — which effectively means that the bridge should return all the contacts.

    * `meta.offset`
        * **Type:** String, optional.
        * **Description:** A generic offset the bridge should apply when retrieving paginated contacts. Can either return page numbers, absolute numerical offsets or some other form of result iteration. This parameter is omitted in the first call and is only sent if the bridge instructs the synchronizer that there are more records available.

    * `meta.context`
        * **Type:** Value, optional.
        * **Description:** A generic field that the bridge can use to keep transitional state between each paginated contacts fetch request. Can be any acceptable JSON value. This parameter is omitted on the first call and is only sent if the bridge both returns a value and instructs the synchronizer that there are more records available.

### Example

```json
{
    "auth": {
        "username": "john.doe@example.com",
        "password": "605b32dd"
    },
    "meta": {
        "offset": "2",
        "synchronization_checkpoint": "2012-11-20 23:01:00 UTC",
        "context": null
    }
}
```

## Steps

1. Use the "auth" fields to authenticate on behalf of the user within the external service. Talkdesk makes sure that all mandatory fields were filled as expected.

2. Depending of the `synchronization_checkpoint` and `offset` parameters within "meta", the bridge might be in one of the following contact retrieval situations:

    > * If no `synchronization_checkpoint` nor the `offset` are sent, this corresponds to an initial sync situation and the bridge should retrieve all the contacts.  
    > * If an `offset` is sent but no `synchronization_checkpoint`, it means that there are pages remaining to be retrieved for the initial sync, so the bridge should get the next page of contacts.  
    > * If an `synchronization_checkpoint` is sent but no `offset`, this corresponds to an incremental sync where only contacts updated after the `synchronization_checkpoint` should be retrieved.  
    > * If both the `synchronization_checkpoint` and an `offset` are sent, this means there are pages remaining to be retrieved for the incremental sync, so the bridge should get the next page of contacts.

3. Make a request to the external service and convert the result to Talkdesk's contact format, as displayed in the response below.

4. Set the `synchronization_checkpoint` to mark the last synchronized contact. Next time Talkdesk synchronizes contacts, it will send along this value, so that the bridge can only retrieve contacts that were updated after this checkpoint. Talkdesk is completely agnostic to the meaning of this field; bridges are responsible for selecting the best possible fit and interpreting it.

5. Set the `next_offset` field to the value of the next page of results when there are more contacts to retrieve. This will instruct Talkdesk's synchronizer that it needs to make another request to the bridge to retrieve the remaining contacts. As with `synchronization_checkpoint`, the system is agnostic to the meaning of this field; bridges can either return page numbers, absolute numerical offsets or some other form of result iteration.

6. The contacts should be sorted in ascending order of the field being used as a checkpoint, to ensure that Talkdesk has indeed synchronized all contacts from the external service until the moment characterized by the checkpoint. Next requests can thus be incremental, using a _get all contacts updated after X_ semantics.

## Response

### Reference

* `next_offset`
    * **Type:** String, optional.
    * **Description:** Value for next page offset, if there are more records that match the query.

* `synchronization_checkpoint`
    * **Type:** String, optional.
    * **Description:** Synchronization checkpoint for this batch of results, resulting in an incremental request next time Talkdesk contact synchronization runs. If not set, all contacts from the external service will be retrieved on every synchronization.

* `context`
    * **Type:** Value, optional.
    * **Description:** Optional internal state to be sent in the next synchronizer request, if `next_offset` is set.

* `contacts`
    * **Type:** Array
    * **Description:** An array of hashes containing contacts ascendingly by the field the bridge chose to mark as the `synchornization_checkpoint`

    * `contacts.<element>.id`
        * **Type:** String, unique, mandatory
        * **Description:** Contact's id in the external service

    * `contacts.<element>.name`
        * **Type:** String, mandatory
        * **Description:** Contact's full name

    * `contacts.<element>.company`
        * **Type:** String, optional
        * **Description:** Contact's company

    * `contacts.<element>.title`
        * **Type:** String, optional
        * **Description:** Contact's title within his company

    * `contacts.<element>.emails`
        * **Type:** Array, optional
        * **Description:** An array of Strings containing contact's emails

    * `contacts.<element>.phones`
        * **Type:** Array, optional
        * **Description:** An array of Strings containing contact's phone numbers

    * `contacts.<element>.photo_url`
        * **Type:** String, optional
        * **Description:** URL location of a contact's photo

    * `contacts.<element>.address`
        * **Type:** String, optional
        * **Description:** Contact's full address

    * `contacts.<element>.websites`
        * **Type:** Array, optional
        * **Description:** An array of Strings containing contact's websites

    * `contacts.<element>.twitter`
        * **Type:** String, optional
        * **Description:** Contact's Twitter username

### Example

```json
{
    "next_offset": "3",
    "synchronization_checkpoint": "2012-12-01 11:21:00 UTC",
    "context": null,
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
            "emails":      ["joe@example.com"]
        }
    ]
}
```
