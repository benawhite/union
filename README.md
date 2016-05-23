# XeJS
Xe is a very small library for connecting isolated JavaScript components together. ~1200 bytes gzipped

Think "Bluetooth for JavaScript"

Typically, a JavaScript library is designed to do one thing very well. (view rendering, storage, communication, logging, etc...) But what if…
* You need to replace your storage library with a different implementation?
* You need to use multiple view libraries at the same time?
* You need your application logic to run on multiple platforms without rewriting the whole thing?

Xe makes it much easier to accomplish these tasks without impacting your entire application. By adding simple rules, an API can be defined for communicating between libraries. This provides a consistent interface that can be adapted for each library. It also provides a simple way to peak into your application internals for analytics, performance tracking, etc…

# API
**xe.initialize(config)**
> Before any interfaces are connected, xe should first be initialized with default interface methods, options, and configutation properties. 
~~~javascript
xe.initialize({
  config: {
    logging: {'*': {minLevel: 'error'}}
  },
  methods: {
    log: function () { console.log.apply(console, arguments); }
  },
  options: {
    isolate: false, // don't clone message
    logging: true // emit errors (true, warn, error, none, false)
  }
});
~~~

**xe.join(interfaceConfig) -> interfaceChannel**
> Each logical component of the application connects to xe using the **join** method. When a component joins xe, an interface channel is returned. Xe does not limit the number of components that can join, even if they have the same role. For example, it is possible for multiple "view" components to join xe.
~~~javascript
var channel = xe.join({
  role: 'view',
  name: 'htmlView' // optional
});
~~~
The channel returned will have **on**, **emit**, and **destroy** methods for communicating with xe. Additionally, it will have any methods that were defined in the **xe.initialize** configuration methods. The above initialize call would have added a **log** method to the channel.

**xe.addRule(role, type, rule)**
> Rules are the backbone of xe. They have the ability to set message routing, manipulate message content, and effectively define the API which all interfaces must conform to. Any messages that are emitted on a channel without a cooresponding rule will be dropped.
~~~javascript
xe.addRule('*', 'log', function () {
    this.toRole = 'logging';
});
~~~
This example routes messages emitted with (**any** interface role) and (**log** message type) to all interfaces with a role of **logging**.

When a channel.emit is executed, the event is immediately passed to the appropriate rules for processing. The rules are selected based on the interface role and message type defined in when the rule is added. If a rule should execute regardless of the interface role, an asterisk * can be used. Asterisks can also be used for the message type as well.

When executed, the rule context will be an event object that has the following read-only properties
* **fromRole** - {string} The role of the interface that emitted the message
* **fromId** - {string} The unique id of the interface that emitted the message

Additionally, these writable properties can be set/updated to control routing and meet other API requirements
* **toRole** - {string} The role of the interfaces the message should be routed to
* **toId** - {string} The unique id of the interface the message should be routed to
* **type** - {string} The type of the message emitted
* **message** - {*} The actual message object emitted
* **return** - {boolean/function} The channel.emit response array will include responses from the routed interfaces. If function, the responses will be passed to the function which should return a single value. e.g. this.return = function (results) { return results.reduce(someFunc)}
 
Methods:
* **connect** - A special method designed for setting up high speed direct connections between interfaces

Here are some other things to consider
* Messages will never be sent back to the interface that originally emitted them. (even if the sender is a valid route target)
* It IS possible for multiple rules to process the same event.
* Each rule is given a fresh event object that it can update in its own way and route however it likes.
* Rules are not required to do anything to the event object. (if the toRole or toId properties are not set, the event will simply be dropped)
* Rules will be executed in the order of specific to generic. e.g. [role, type] -> [role, * ] -> [ * , type] -> [ * , * ]
* Return values from toId targets will be added to the beginning of the response array
* Return values from toRole targets will be added to the end of the response array
* If no interface handlers are found for a given message the response array will be empty

# Errors
When xe encounters an error processing a message, it will emit its own internal message that can be listened for. The message emitted will use and interface role of "!nternal".
Errors will be thrown in the following circumstances:

**error type**
* exception thrown inside interface listener
* exception thrown inside mutator function

**warn type**
* message dropped: no mutators found
* message dropped: no listeners found

**Message Structure**
~~~javascript
{
  error: {Error}, // the error instance
  from: {String}, // the source where the emit originated from (inferfaceRole/interfaceId)
  to: {String[]}, // an array of targets where the message was trying to be sent (inferfaceRole/* or interfaceId)
  type: {String}, // the emitted message type
  message: {Object} // the original emitted message
}
~~~

**Example Rules**
~~~javascript
hub.addMutator('!nternal', 'error', function (event) {
    console.error(event.message);
});
hub.addMutator('!nternal', 'warn', function (event) {
    console.warn(event.message);
});
hub.addMutator('!nternal', '*', function (event) {
    console[event.type] && console[event.type](event.message);
});
~~~
