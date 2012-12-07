Talkdesk Integrations API makes it easy for third-parties to send data to and from Talkdesk. The system is designed to work asynchronously through HTTP web-hooks. Third parties create their own integrations by providing Talkdesk a specific configuration and implementing their own bridges.

## Concepts

A _bridge_ is an intermediary HTTP application that maps structured data living in a third-party system to a streamlined format that Talkdesk can process for consumption.

```
Diagram goes here
```

The bridge can, therefore, adapt any system that holds valuable business data that can be displayed by Talkdesk in the form of a contact or interaction. This can be anything the bridge can retrieve and send data to (such as an Internet service, an internal enterprise system or even a database) and will, from now on, be referenced as the _external service_.

## Usage

One of the main usages for the Integration API is to enrich Talkdesk's contacts with related business information from other sources.

Depending on the type of external service being integrated, the bridge may need to retrieve information from it on behalf of the user. When this is the case, Talkdesk will let the integration configure which authentication fields it needs from the user in order to talk to the external service. Talkdesk will then store these and send them along in every future request it will make to the bridge.

The asynchronous nature of this API makes it possible to choose which features the bridge provides by only configuring a subset of the supported endpoints. Below are the API features a bridge can be configured to hook into through an HTTP endpoint:

* __Authorization validation:__ When users activate an integration, Talkdesk will retrieve their credentials and send those through an HTTP request to the bridge's configured endpoint. If considered valid, these credentials will be passed along in all further requests for this user, making it possible for the bridge to authenticate with the external service when retrieving actual data.

* __Contact synchronization:__ Talkdesk will use the configured bridge's endpoint to periodically retrieve contacts from the external service, making them available in the user's Talkdesk account.

* __Interaction retrieval:__ An integration can also provide an interaction update endpoint. Talkdesk will use it when a user visits a contact in the interface to retrieve interactions with that contact from the external service.

* __Action execution:__ It is also possible for bridges to configure actions to be executed in the external service. These can be invoked by Talkdesk when some events happen and can be used to make data flow to the external service. These can have different scopes depending on where they're available to the user inside Talkdesk, and are also configured through web-hook endpoints within the bridge.

## Developing

All you have to do to integrate with Talkdesk is to develop an HTTP web service that exposes one or more of these endpoints through the following JSON configuration:

* [Configure an integration](integrations/configuration.md)

The Integration API defines a JSON data format for the requests being made to the bridge and expected responses for each of the endpoints:

* [Create a contact retriever](integrations/contact-synchronization.md)
* [Create an interaction retriever](integrations/interaction-updatemd.md)
* [Create an action executor](integrations/action-execution.md)
* [Create an authorization validator](integrations/auth-validation.md)
