Synket
======

Synket allow synchronous sockets. The flow of operations is as follows:

1. Client sends a message to the socket

2. Client is paused

3. Server receives the message 

4. Server does some processing that can take any length of time

5. Server sends data back to Client

6. Client is resumed and the return value of the the `send` command is the data that was written to the socket by Server



Firstly, a basic server server running on another thread on the same machine or another machine:


```js
const net = require('net');

var server = net.createServer(function(sock) {
	socket.setEncoding('utf8');

	//Receive data from client
	sock.on('data', function(data) {

		///Do something that takes time:
		
		setTimeout(function() {
			sock.write('The client sent: ' + data);
		}, 3000);
	});

	
}).listen({path: './test.sock'});

/*
//Alternatively listen on a URL and port:

}).listen({url: 'http://example.org', port: 4564});

*/

```

Now the client:


```js
const Synket = require('synket');

//start the socket
var connection = new Synket({path: './test.sock'});

/*
//Alternatively use a URL and port:

var connection = new Synket({url: 'http://example.org', port: 4564});
*/

//send a message, will not return until data is written to the socket by the host
var result = connection.send('doSomething');

console.log(result);
```


When this code runs it pause for 3 seconds then print `The client sent: doSomething`.


Quick Server
------------

Synket also provides a mechanism for launching a server that requires less bootstrap code. 

You can write a server as node module that returns a class with methods e.g. `MyServer.js`:

```js

module.exports = class {
	doSomething(socket, args) {
		setTimeout(function() {
			setTimeout(function() {
				socket.write('doSomething called with ' + args[0] + ' and ' + args[1]);
			}, 3000);
		});
	}
}
```

And then launch it from a node script (this can be the client or another script):

```js
const Synket = require('synket');

//start the socket
var connection = new Synket({path: './test.sock'});

/*
//Alternatively use a URL and port:

var connection = new Synket({url: 'http://example.org', port: 4564});
*/


//Start the server stored in `MyServer.js` this will pause execution until the server is up
connection.startServer('./MyServer.js');

```

The server will now be launched in its own thread and you can send messages to it like so:

```js
var result = connection.send('doSomething', ['foo', 'bar']);

console.log(result);
```

This will print `doSomething called with foo and bar`.



FAQ
===

**Does blocking cause a large overhead?**

In short, no. Synket does not use a `while` loop to block. The execution of the client thread is paused until a message back from the server is received. This is not much different to leaving the thread waiting asynchronously for a response. Synket does have to launch a third node process to act as a middle-man which will add a small overhead, however this overhead is negligible as it runs a 20 line js file in another process.

