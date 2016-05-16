# union
Union is a very small library that brings interchangeable JavaScript components together. ~1200 bytes gziped

**union.initialize(config)**
> Before any interfaces are connected, the union should first be initialized with default interface methods, options, and configutation properties. 
~~~javascript
union.initialize({
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

**union.join(interfaceConfig) -> interfaceChannel**
> Each logical component of the application connects to the union using the **join** method. When a component joins the union, an interface channel is returned. The union does not limit the number of components that can join, even if they have the same role. For example, it is possible for multiple "view" components to join the union.
~~~javascript
var channel = union.join({
  role: 'view',
  name: 'htmlView' // optional
});
~~~
The channel returned will have **on**, **emit**, and **destroy** methods for communicating to the union. Additionally, it will have any methods that were defined in the **union.initialize** configuration. The above initialize call would have added a **log** method to the channel.

**union.addRule(role, type, rule)**
> Rules are the backbone of the union. They have the ability to set message routing, manipulate message content, and effectively define the API which all interfaces must conform to. Any messages that are emitted on a channel without a cooresponding rule will be dropped.
~~~javascript
union.addRule('*', 'log', function () {
    this.toRole = 'logging';
});
~~~
This example routes messages emitted with (**any** interface role) and (**log** message type) to all interfaces with a role of **logging**.
