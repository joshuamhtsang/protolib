# Josh's Notes on protolib

## Attempt to create Docker image with protolib and a protoApp compiled inside

Dockerfile for building the protoCapExample created.  This protoApp listens and captures packets on a network interface, see the cpp file here:
[protoCapExample.cpp](./examples/protoCapExample.cpp)


~~~
$ docker build -f Dockerfile_1_protoCapExample --progress=plain --no-cache -t protolib_proto-cap-example .
~~~

~~~
$ docker run -it --name protoCapExample-a protolib_proto-cap-example bash
~~~

The C++ application is contain within the `build/` directory within Docker container:

~~~
/app/protolib/build/
~~~

And you can run the examples you've built specified to build:

~~~
root@xxx:/app/protolib/build# ./protoCapExample 
protoCapExample: got interface index:2 for interface: 172.17.0.2
              (real interface name: "eth0")
              (interface MAC addr: "16:5f:22:1c:33:04")
Proto Error: protoCapExample: Done.
~~~

To listen to a particular interface (say, `eth0`):
~~~
root@xxx:/app/protolib/build# ./protoCapExample listen eth0
protoCapExample: got interface index:2 for interface: 172.17.0.2
              (real interface name: "eth0")
              (interface MAC addr: "46:47:1c:ae:cf:fb")
protoCapExample: listening to interface: eth0...
stats: rcvd>1 sent>0 serr>0
stats: rcvd>2 sent>0 serr>0
stats: rcvd>3 sent>0 serr>0
~~~


## Testing the protoCapExample: tcp connection between 2 containers

An illustrative test of the `protoCapExample` protoApp.  When run, this app listens on an interface and can also forward to another/same interface.

The setup is shown in the diagram below.  Container `protoCapExample-a`  (container A) will run a netcat server and the `protoCapExample` on a combination of listen and forward modes.  Container `protoCapExample-b` (Container B) will start a netcat client.

┌─────────────────────────────────────────────────┐
│                   host                          │
│               (172.17.0.1)                      │
└─────────────────────────────────────────────────┘
                                                               
┌─────────────────────────────────────────────────┐
│        Container A : protoCapExample-a          │
│               (172.17.0.2)                      │
│                                                 │
│   $ nc -l -p 8888                               │
│   $ ./protoCapExample listen eth0               │
│                 or                              │
│   $ ./protoCapExample listen eth0 forward eth0  │
└─────────────────────────────────────────────────┘
                                                   
┌─────────────────────────────────────────────────┐
│        Container B : protoCapExample-b          │
│                (172.17.0.3)                     │
│                                                 │
│   $ nc 172.17.0.2 8888                          │
└─────────────────────────────────────────────────┘

Diagram created with: https://asciiflow.com/#/

Note the 172.17.0.0/16 network is set up by Docker, and containers are added to this network by default.

Start the containers in separate terminals, 2 terminals for Container A:

~~~
$ docker run -it --name protoCapExample-a protolib_proto-cap-example bash
~~~
~~~
$ docker exec -it  protoCapExample-a bash
~~~

And 1 terminal for Container B:

~~~
$ docker run -it --name protoCapExample-b protolib_proto-cap-example bash
~~~

Run the corresponding CLI commands shown in the diagram.  Note how when protoCapExample has 'forward' enabled, anything transmitted over netcat from Container B to A is sent back, evident by the 'rcvd' and 'sent' packet counts both increasing.

When protoCapExample is just listening on eth0:
~~~
stats: rcvd>0 sent>0 serr>0
stats: rcvd>2 sent>0 serr>0 (+2 to rcvd!)
~~~

When protoCapExample is both listening and forwarding on eth0:
~~~
stats: rcvd>0 sent>0 serr>0
stats: rcvd>2 sent>1 serr>0 (+2 to rcvd and +1 to sent!)
~~~