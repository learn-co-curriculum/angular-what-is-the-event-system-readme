# What is the event system?

## Overview

Another powerful offering from Angular is the integrated event system. This allows us to publish and subcribe to events. We've actually used this already - remember using `$scope.$on` to listen to the `$destroy` event?

## Objectives

- Describe the publish/subscribe event pattern
- Describe the event system

## Publishing events

All events can be published on our `$scope` or `$rootScope` objects.

Angular offers us two ways of publishing events - either up or down. Up will go all the way from the current scope to our root scope, and down will go down from our current scope into it's children scopes, and all it's children's scopes, and so on and so forth.

To publish events downwards, we use `$scope.$broadcast`. To publish event upwards, we use `$scope.$emit`.

The first argument we pass through to these functions are the name of the event. This is what we would then specify when we want to listen for the event.

The second argument we pass through is data. This can then be picked up by the subscriber. For instance, we might want to publish an event when the user sends a message - we can send the message data through with the event too, and subscribe to it in a directive that then displays the message.

Examples of this are:

```js
$scope.$emit('eventName'); // passing through data is optional

$scope.$emit('anotherEvent', { obj: 'hello!' });

$scope.$broadcast('aDifferentEvent', 3949324); // we can pass through any data
```

## Subscribing to events

We can then use a function named `$on` to subscribe to these events. We pass through a callback too, which retrieves the data (if there is any) too.

We can subscribe to the events above like so:

```
$scope.$on('eventName', function () {
	// no data passed through
});

$scope.$on('anotherEvent', function (event, data) {
	// data = { obj: 'hello' }
});

$scope.$on('aDifferentEvent', function (event, data) {
	// data = 3949324
});
```

Simple!

## $rootScope

We can also broadcast events on the `$rootScope` - after all it is a scope! As all the children scopes are *eventual* children of the root scope, we will use `$broadcast` as this goes down the scopes.

Publishing and subscribing to events on the `$rootScope` is generally preferred - most of the time there is no advantage to only publishing events upwards/downwards from the current scope so by using events on the root scope, we can guarantee everyone will receive the event we are broadcasting.

```js
$rootScope.$emit('eventName');
$rootScope.$broadcast('aDifferentEvent', 3949324);

$rootScope.$on('eventName', function () {
	// awesome!
});
$rootScope.$on('aDifferentEvent', function (event, data) {
	// awesome!
});
```

## Unsubscribing from an event

One awesome thing that Angular does when we subscribe to an event is provide us a really easy way to unsubscribe from it. Angular will automatically unsubscribe all the event subscribers on our `$scope` object (that's how we can use `$scope.$on` to listen for `$destroy`), but it can't unsuscribe from our subscribers on the `$rootScope`.

`$scope.$on` returns a closure function that we can actually call to unsubscribe it!

```js
var unbind = $rootScope.$on('eventName', function () {
	// awesome!
});

// unbind === func() {}
```

We can then call `unbind()`, and this will unsubscribe the event!

We could then hook this into our `$scope.$on('destroy')` function call:

```js
var unbind = $rootScope.$on('eventName', function () {
	// awesome!
});

$scope.$on('$destroy', unbind);
```