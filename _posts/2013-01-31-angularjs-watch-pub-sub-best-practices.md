---
layout: default
title: angularjs.org watch, on pub sub, and you.
excerpt: A team I work on has been using angularjs.org for more than half a year now, and I wanted to put out some best practices...
---
A team I work on has been using [angularjs.org's](http://angularjs.org) various watch and event systems for more than half a year now, and I wanted to put out some best practices that we've learned from painful experience.

### When to use a watch:
* when you're writing a directive and want data binding behavior
* when you're watching a stateful service 
* when it's OK for the ui and state update to happen in a non-deterministic order

### When NOT to use a watch:
* when you need to orchestrate complicated data loading patterns
* when you're watching the routeParams.

## Best practices around watches:

* check for newValue !== oldValue in the watch callback.  Watch callbacks are always called at least once.
watch functions, not properties.  You can't set breakpoints on properties

* if your watcher function looks like this:

{% highlight javascript %}
$scope.$watch(function(){return SomeService.stateValueGetter()},
    function(newValue,oldValue){});
{% endhighlight %}

just do this:
{% highlight javascript %}
$scope.$watch(SomeService.stateValueGetter,
    function(newValue,oldValue){});
{% endhighlight %}

### When to use pub/sub:

* when you need to let multiple subscribers know about an event and those subscribers need to do more than radiate information to their view.

### When NOT to use pub/sub:

* when you're watching a simple state based service and using that to update a view.
* when you're writing a general purpose directive and need data binding.

## Best practices around pub/sub:

* encapsulate subject-specific watches using the following pattern:

{% highlight javascript %}
// an example channel service that lets consumers
// subscribe and publish for nuclear reactor meltdowns
 
var CoreReactorChannel = function($rootScope) {
 
    // local constants for the message ids.  
    // these are private implementation detail
    var ELEVATED_CORE_TEMPERATURE_MESSAGE = "elevatedCoreMessage";
 
    // publish elevatedCoreTemperature
    // note that the parameters are particular to the problem domain
    var elevatedCoreTemperature = function(core_id, temperature) {
        $rootScope.$broadcast(ELEVATED_CORE_TEMPERATURE_MESSAGE,
            {
                core_id: core_id,
                temperature: temperature
            });
    };
 
    // subscribe to elevatedCoreTemperature event.
    // note that you should require $scope first 
    // so that when the subscriber is destroyed you 
    // don't create a closure over it, and te scope can clean up. 
    var onElevatedCoreTemperature = function($scope, handler) {
        $scope.$on(ELEVATED_CORE_TEMPERATURE_MESSAGE, function(event, message){
            // note that the handler is passed the problem domain parameters 
            handler(core_id, temperature);
        });
    };

    // other CoreReactorChannel events would go here.

    return {
        elevatedCoreTemperature: elevatedCoreTemperature,
        onElevatedCoreTemperature: onElevatedCoreTemperature
    };
};

 
//Usage elsewhere in the application
var MyController = function($scope, CoreReactorChannel){
    // Handle core temperature changes
    var onCoreTemperatureChange = function(message){
    console.log(message.core_id, message.temperature);
    }
    
    // Let the CoreReactorChannel know the temperature has changed
    $scope.changeTemperature = function(core_id, temperature){
        CoreReactorChannel.elevatedCoreTemperature(core_id, temperature);
    }
 
    // Listen for temperature changes and call a handler
    // Note: The handler can be an inline function
    CoreReactorChannel.onElevatedCoreTemperature($scope, onCoreTemperatureChange);
};   
{% endhighlight %}

* don't mix concerns in the channel code.  A general channel is a ball of mud.
