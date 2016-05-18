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
    logging: true // emit errors
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
The channel returned will have **on**, **emit**, and **destroy** methods for communicating with xe. Additionally, it will have any methods that were defined in the **xe.initialize** configuration. The above initialize call would have added a **log** method to the channel.

**xe.addRule(role, type, rule)**
> Rules are the backbone of xe. They have the ability to set message routing, manipulate message content, and effectively define the API which all interfaces must conform to. Any messages that are emitted on a channel without a cooresponding rule will be dropped.
~~~javascript
xe.addRule('*', 'log', function () {
    this.toRole = 'logging';
});
~~~
This example routes messages emitted with (**any** interface role) and (**log** message type) to all interfaces with a role of **logging**.
