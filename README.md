# What is the event system?

## Overview

Another powerful offering from Angular is the integrated event system. This allows us to publish and subcribe to events. We've actually used this already - remember using `$scope.$on` to listen to the `$destroy` event?

## Objectives

- Describe the publish/subscribe event pattern
- Describe the event system

## Publishing events

All events can be published on our `$scope` or `$rootScope` objects. Why do we use events? Well, communication between controllers in two different aspects of the application can become quite hard - this is where events come in. Or if we receive data in a service and that data gets updated - we can notify the controllers that there is new data to consume.

Angular offers us two ways of publishing events - either up or down. Up will go all the way from the current scope to our root scope, and down will go down from our current scope into it's children scopes, and all it's children's scopes, and so on and so forth.

Child scopes are a bit tricky to understand - but don't worry, they're really simple! Let's imagine where we start our app - `ng-app`. This is our ``$rootScope`. Then, we use `ng-controller` or a directive inside `ng-app`. This will create another scope, inside of our root scope. Then, if we were to put use a directive inside of them, we'd get a child scope inside their scope. Whenever we use a directive (`ng-controller`, `ng-repeat`, custom directives etc) that create their own scope, they're made in their parents scopes.

To publish events downwards, we use `$scope.$broadcast`. To publish event upwards, we use `$scope.$emit`.

The first argument we pass through to these functions are the name of the event. This is what we would then specify when we want to listen for the event. We could have a message being sent and then received, so we'd generally namespace these into `message` and then the action - such as `message:sent` and `message:received`.

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

Publishing and subscribing to events on the `$rootScope` is generally preferred - most of the time there is no advantage to only publishing events upwards/downwards from the current scope so by using events on the root scope, we can guarantee everyone will receive the event we are broadcasting - every scope will get notified of the event.

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

We could then hook this into our `$scope.$on('destroy')` function call (the one that gets emitted when our directive is about to be removed):

```js
var unbind = $rootScope.$on('eventName', function () {
	// awesome!
});

$scope.$on('$destroy', unbind);
```