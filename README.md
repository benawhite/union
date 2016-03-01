# union
Union is a very small library that brings interchangeable JavaScript components together. ~1200 bytes gziped

**union.join(interfaceConfig) -> interfaceChannel**
> Each logical component of the application must connect to the union using the join method. When a component joins the union, an interface channel is returned. The union does not limit the number of components that can join with the same role. For eaxample, it is possible for multiple view components to join the same union. The API must account for these situations.
~~~javascript
var channel = union.join({
  role: 'someRole',
  name: 'myInterface' // optional
});
~~~

**union.addRule(role, type, mutator)**
> Rules are the brains of the union. These functions have the ability to route messages, manipulate the message content, and effectively define the API which all interfaces must conform to.
This example routes all messages emitted with (**any** interface role) and (**log** message type) to all interfaces with a role of **logging**.
~~~javascript
union.addRule('*', 'log', function (event) {
    event.toRole = 'logging';
});
~~~
> Rules are covered in more details below.
