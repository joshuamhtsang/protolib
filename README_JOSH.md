# Josh's Notes on protolib

## Attempt to create Docker image with protolib compiled inside

Dockerfile created.

~~~
$ docker build --progress=plain --no-cache -t protolib-josh-build .
~~~

~~~
$ docker run -it protolib-josh-build-1 bash
~~~

The C++ application is contain in directory within Docker container:

~~~
/app/protolib/build
~~~

And you can run the examples you've specified to build:

~~~
root@c8c673a60af1:/app/protolib/build# ./protoCapExample 
protoCapExample: got interface index:2 for interface: 172.17.0.2
              (real interface name: "eth0")
              (interface MAC addr: "16:5f:22:1c:33:04")
Proto Error: protoCapExample: Done.
~~~