# docker-exec-websocket-server
##Purpose
A server that serves the results of docker exec over websockets. 

##Usage
Server: 
```js
var DockerServer = require('../lib/server.js');
var dockerServer = new DockerServer({
  path: '/'+slugid.v4(),    //Path to WebSocket
                            //can be randomized to prevent unauthorized access
  port: 8081,               //Port to WebSocket, required
  container: 'servertest',  //Container to inject exec proccess into
});

```
By default, uses `/var/run/docker.sock` to communicate with Docker.

Client: 
```js
var socket = new WebSocket('ws://localhost:<port>/<path>');
socket.onopen = function() {
  socket.send('/bin/bash');  //First message sent indicates command for docker exec
  socket.onmessage = function() {
    //First message sent is a 'ready' message
    //Can start sending input now
    inputStream.pipe(socket);
    socket.onmessage = function(msg) {
      //Can start parsing data now
      outputStream.write(msg.data);
    };
  };
};
```

##Message Types
Messages are prepended with a single byte which contains information about the encoded message.

1: Message was sent through stdout.

2: Message was sent through stderr.

100: Indicates that the process run through exec has exited; payload contains 4 byte little-endian exit code.

##Testing
Docker 1.6.1 or above must be installed with a container named `servertest` running with `cat` capabilities to inject the exec process into. From there, `npm test` will carry out the test.
