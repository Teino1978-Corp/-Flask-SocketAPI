Flask-SocketAPI
===============

Lightweight library to create streaming API over Flask-SocketIO.

Installation
------------

You can download and unzip the latest version of this package from github: [https://github.com/kyouko-taiga/Flask-SocketAPI/archive/master.zip](https://github.com/kyouko-taiga/Flask-SocketAPI/archive/master.zip).

Alternatively, if you have git installed on your system, you may prefer cloning the repository directly:

	git clone https://github.com/kyouko-taiga/Flask-SocketAPI.git

Then, simply navigate to the root directory of the package and run:

	python setup.py install

API protocol
------------

The protocol of the API generated by Flask-SocketAPI is defined as follows:

### About URIs

All resources are identified by special endpoint called URI, which look like any regular URLs.
Like in REST, those that end with a forward slash are called list URIs and are used to identify list of resources.
Let's say we have an URI of the form `/koala/32`. Typically we would say that this URI identifies a resource of type `koala` with identifier `42`. Moreover, we would say that `/koala/` corresponds to a list of `koalas`.

All messages use those URIs to identify what resource is concerned.

### The protocol

A client can patch a resource with a `patch` event, which should be sent with a description of the attributes it wants to patch.

```javascript
socket.emit('patch', {
    uri: <uri>,
    patch: {
        <attribute 1>: <patch 1>,
        <attribute 2>: <patch 2>,
        ...
    }
})
```

A client can create a resource with a `create` event.

```javascript
socket.emit('create', {
    uri: <list uri>,
    attributes: {
        <attribute 1>: <value 1>,
        <attribute 2>: <value 2>,
        ...
    }
})
```

A client can delete a resource with a `delete` event.

```javascript
socket.emit('delete', {
    uri: <uri>
})
```

A client can subscribe to all modification of a resource with a `subscribe` event.

```javascript
socket.emit('subscribe', <uri>);
```

A client can unsubscribe from a resource with a `unsubscribe` event..

```javascript
socket.emit('unsubscribe', <uri>);
```

Once the client subscribed to a resource, the server will send it a `state` event with the current state of the subscribed resource.
After that, it will forward any `patch`, `create` and `delete` events that it receives until the client unsubscribes.

Usage
-----

Flask-SocketAPI defines 4 decorators to help you design your API.
Each decorator takes a route as an argument, which defines the URIs that should be routed to it.
Those routes are identical to those of Flask.

1. `resource_creator`

	This decorator allows you to define resource creators.
	Decorated functions will receive the attributes with their values as sent by the `create` requests from the client, and is expected to return a new instance of the resource it is defined for.

	Here's an example:

	```python
	@socketapi.resource_creator('/todo/')
	def create_todo(**kwargs):
		todo = Todo(**kwargs)
		todo.save()
	    return todo
	```

2. `resource_getter`

	This decorator allows you to define resource getters.
	Decorated functions will receive the arguments of the decorator URI.

	Here's an example:

	```python
	@socketapi.resource_getter('/todo/<id_>')
	def get_todo(id_):
		return Todo.query.filter(Todo.id_ == id_).one_or_none()
	```

3. `resource_patcher`

	This decorator allows you to define resource patchers.
	Decorated functions will receive the attributes with their values as sent by the `patch` request from the client, as well as the arguments of the decorator URI.
	It is expected patch the resource accordingly.

	Here's an example:

	```python
	@socketapi.resource_patcher('/todo/<id_>')
	def patch_todo(id_, patch):
		todo = Todo.query.filter(Todo.id_ == id_).one()
	    for attribute, value in patch.items():
	        setattr(todo, attribute, value)
		todo.save()
	```

	Note that a patcher can make various modifications on the patch arguments before it actually updates the resource, so as to handle side effects for instance.

	Moreover, note you can define multiple patchers for the same URI.
	They will be called in the order they were defined.

4. `resource_deleter`

	This decorator allows you to define resource deleters.
	Decorated functions will receive the arguments of the decorator URI.

	Here's an example:

	```python
	@delete_todo('/todo/<id_>')
	def delete_todo(id_):
		todo = Todo.query.filter(Todo.id_ == id_)
		todo.delete()
	```

Examples
--------

You can check fully working examples in the `examples/` directory.
