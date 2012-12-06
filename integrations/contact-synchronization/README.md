## Implementing a Contact Retriever

Talkdesk will send an HTTP POST to the bridge's configured endpoint when a contact synchronization is run. This request can be slightly different depending on the situation for which it is run:

* __Initial Sync:__ The first time the contact synchronization runs for a given account, no `synchronization_checkpoint` is sent as a "meta" field, meaning that this is a full retrieval of every contact in the external system.

* __Incremental Sync:__ When a contact sync has been made, the bridge can set a `synchronization_checkpoint` in the response so that the next time the bridge is asked to sync contacts it will only retrieve the ones that changed after the checkpoint. Talkdesk will use the `synchornization_checkpoint` "meta" field for that purpose.

* __Next Page:__ For the previous cases, when adapting third-party services that return contacts in pages, the bridge can signal Talkdesk that the returned contacts are partial by setting `next_offset` in the response. If this is the case, Talkdesk will make a new request for every next page by setting the `offset` "meta" field, until all contacts are retrieved, thus no `next_offset` being returned.

### Request

```json
{
    "auth": {
        <auth_field>: <auth_field_value>,
        <auth_field>: <auth_field_value>,
        ...
    },
    "meta": {
        "synchronization_checkpoint": <synchronization_checkpoint>,
        "offset": <offset>
    }
}
```

### Steps

1. Process the "auth" request fields to configure own external system client authorization parameters.

2. Process `synchronization_checkpoint` and `offset` "meta" fields to prepare request to be made to the external system:

    * If no `synchornization_checkpoint` is sent, do a full contact retrieval.
    * If no `offset` is sent, retrieve the first page of contacts.

3. Make a request to the external system and convert the result to Talkdesk's contact format, as displayed in the response below.

4. Return contacts in ascending order, sorted by whatever field you deem as useful to use as a checkpoint.

5. Set the `synchronization_checkpoint` to the value of the last contact's field you chose as a checkpoint. Talkdesk's contact synchronization system is completely agnostic to the meaning of this field; bridges are responsible for selecting the best possible fit and interpreting it.

6. Set the `next_offset` field to the value of the next page of results when there are more contacts to retrieve. This will instruct Talkdesk's synchronizer that it needs to make another request to the bridge to retrieve the remaining contacts. As with `synchronization_checkpoint`, the system is agnostic to the meaning of this field; bridges can either return page numbers, absolute numerical offsets or some other form of result iteration.

### Response

```json
{
  "synchronization_checkpoint": <synchornization_checkpoint>,
    "next_offset": <next_offset>,
    "contacts": [
        {
            "id":          <contact id in the external system>,
            "name":        <contact's name>,
            "title":       <contact's title>,
            "emails":      <array of emails (strings)>,
            "phones":      <array of phones (strings)>,
            "photo_url":   <url with the photo of the contact>,
            "company":     <company name>,
            "address":     <contact's address>,
            "websites":    <array of websites (strings)>,
            "twitter":     <contact's twitter username>
        }
    ]
}
```
