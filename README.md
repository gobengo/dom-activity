# dom-activity

Publish and Subscribe to Activities that occur in a web document.

## Status of this Document

A sketch to provoke reactions from others. Everything that follows (and precedes) should change according to community consensus. Please file issues if you are offended or inspired.

This is a proposal Web API for publishing Web [Activity](http://www.w3.org/TR/2015/WD-activitystreams-vocabulary-20150129/#dfn-activity) events in a web document in a standard way such that other agents in the document can react to these activities.

## Use Cases

* As a user, I can configure my user-agent to send my activity to a personal [ActivityPump](http://w3c-social.github.io/activitypump/) Web Service
* As a website operator, I can write a handler for all activities performed by my end-users interacting with sections of my webpages authored by me, my organization, and vendors that provide code and services that I run on my webpage
* A user Creates an Open Web Annotation
  - alternative proposal for this use case: [DOMAnnotations](https://docs.google.com/presentation/d/1ZKSZyLbEFqwYaG32cFJwcXQjjiU2ZzQguDmyhKfaTdY/edit#slide=id.g99ebca8a5_1_46) (though dom-activity is a generalization)

## Dependencies

* [What is an Activity?](http://www.w3.org/TR/2015/WD-activitystreams-vocabulary-20150129/#dfn-activity)
* What are some example [Activity Types](http://www.w3.org/TR/2015/WD-activitystreams-vocabulary-20150129/#activity-types)?
* What are some standard [Object Types](http://www.w3.org/TR/2015/WD-activitystreams-vocabulary-20150129/#object-types) that would make up an Activity Actor, Object, or Target?
* What are some common [Properties](http://www.w3.org/TR/2015/WD-activitystreams-vocabulary-20150129/#properties) that Activities and Objects frequently have?
* How can I extend the standard Activity and Object Types with [more Properties](http://www.w3.org/TR/json-ld/#dfn-property)?

## Provides

### Standards

* How do I publish an Activity that ocurred in a Node tree that I control?
  1. Create an ActivityEvent
    - userland convention
    ```js
    let exampleActivityEvent = new CustomEvent('activity', {
      detail: { actor, verb, object }
    })
    ```
    - Standard straw man
    ```js
    let exampleActivityEvent = new ActivityEvent({
      activity: { actor, verb, object }
    })
    ```
    - future-proof userland convention
    ```js
    let exampleActivityEvent = (function (activity) {
      let event = new CustomEvent('activity', {
        detail: {activity}
      });
      event.activity = activity;
      return event;
    }({ actor, verb, object}))
    ```
  2. Dispatch it
  ```js
  (document.querySelector('#activity-source') || document)
    .dispatchEvent(exampleActivityEvent)
  ```

* How can I react to all Activity occurring in the document?
  - future-proof userland convention
  ```js
  document.addEventListener('activity', function (e) {
    let activity = e.activity || (e.detail ? e.detail.activity : undefined);
    if ( ! activity) return;
    // handle the activity
  })
  ```
  - Standard straw man
  ```javascript
  (new ActivityStream(document)).pipeTo(
    new WritableStream({
      start(error) {},
      write(activity) {
        console.log('There was an activity in the document', activity)
      },
      close() {
        console.log('Called if the eventTarget can no longer dispatch ActivityEvents (e.g. the Element is garbage collected)')
      }
    });
  )
  .then(() => console.log("All data successfully written!"))
  .catch(e => console.error("Something went wrong!", e));
  ```
    - ReadableStream provided by [WHATWG Streams](https://streams.spec.whatwg.org/#rs-class)
    - Perhaps a `new EventStream(eventTarget, [types])` constructor would be useful as a generalization

### Examples

* What sort of [Actor](http://www.w3.org/TR/2015/WD-activitystreams-vocabulary-20150129/#dfn-actor)s create activity in a document?
  - [Application](http://www.w3.org/TR/2015/WD-activitystreams-vocabulary-20150129/#dfn-application)s (e.g. web components) that load on the page, and publish lifecycle activity
  - [Identities](http://www.w3.org/TR/2015/WD-activitystreams-vocabulary-20150129/#dfn-identity)
    + e.g. `http://www.w3.org/ns/activitystreams#Anonymous`
    + e.g. Identities with [Credentials](https://w3c.github.io/webappsec/specs/credentialmanagement/#interfaces-credential-types-credential)
      * e.g. [livefyre-auth](https://github.com/Livefyre/livefyre-auth) credentials
* What sort of verbs create activity in a document?
  - TODO with full list of real-world examples, where 'real-world' means [bengo.is](https://bengo.is) is implementing them like now
  - 'appLoaded'
  - 'appBecomeVisible'
  - 'clicked' subelement resource
  - 'hovered' on subelement resource
* What might I want do with all the activity in a document?
  - Send to the web server to understand user behavior to provide a better experience
  - Send to an analytics vendor
  - As a user, I can configure my user-agent to send my activity to a personal [ActivityPump](http://w3c-social.github.io/activitypump/) Web Service
  - TODO for you: What would you do?

