---
bodyclass: docs
layout: docs
headline: Dart Quickstart
sidenav: doc-side-quickstart-nav.html
type: markdown
---
<p class="lead">This guide gets you started with gRPC in Dart with a simple
working example.</p>

<div id="toc"></div>

### Prerequisites

#### Dart SDK

gRPC requires Dart SDK version 2.0 or higher. Dart gRPC supports Flutter and Server platforms.

For installation instructions, follow this guide: [Install Dart](https://www.dartlang.org/install)

#### Install Protocol Buffers v3

While not mandatory to use gRPC, gRPC applications usually leverage Protocol
Buffers v3 for service definitions and data serialization, and our example code
uses Protocol Buffers as well as gRPC.

The simplest way to install the protoc compiler is to download pre-compiled
binaries for your operating system (`protoc-<version>-<os>.zip`) from here:
[https://github.com/google/protobuf/releases](https://github.com/google/protobuf/releases)

  * Unzip this file.
  * Update the environment variable `PATH` to include the path to the protoc
    binary file.

Next, install the protoc plugin for Dart

```sh
$ pub global activate protoc_plugin
```

The compiler plugin, `protoc-gen-dart`, is installed in `$HOME/.pub-cache/bin`.
It must be in your $PATH for the protocol compiler, protoc, to find it.

```sh
$ export PATH=$PATH:$HOME/.pub-cache/bin
```

## Download the example

You'll need a local copy of the example code to work through this quickstart.
Download the example code from our GitHub repository (the following command
clones the entire repository, but you just need the examples for this quickstart
and other tutorials):

```sh
$ # Clone the repository at the latest release to get the example code:
$ git clone https://github.com/grpc/grpc-dart
$ # Navigate to the "Hello World" Dart example:
$ cd grpc-dart/example/helloworld
```

## Run a gRPC application

From the `example/helloworld` directory:

1. Download package dependencies

   ```sh
   $ pub get
   ```

2. Run the server

   ```sh
   $ dart bin/server.dart
   ```

3. In another terminal, run the client

   ```sh
   $ dart bin/client.dart
   ```

Congratulations! You've just run a client-server application with gRPC.

## Update a gRPC service

Now let's look at how to update the application with an extra method on the
server for the client to call. Our gRPC service is defined using protocol
buffers; you can find out lots more about how to define a service in a `.proto`
file in [gRPC Basics: Dart][]. For now all you need to know is that both the
server and the client "stub" have a `SayHello` RPC method that takes a
`HelloRequest` parameter from the client and returns a `HelloReply` from the
server, and that this method is defined like this:


```
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```
Let's update this so that the `Greeter` service has two methods. Edit
`protos/helloworld.proto` and update it with a new `SayHelloAgain`
method, with the same request and response types:

```
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

(Don't forget to save the file!)

## Generate gRPC code

Next we need to update the gRPC code used by our application to use the new
service definition.

From the `example/helloworld` directory, run:

```sh
$ protoc --dart_out=grpc:lib/src/generated -Iprotos protos/helloworld.proto
```

This regenerates the files in `lib/src/generated` which contain our generated
request and response classes, and client and server classes.

## Update and run the application

We now have new generated server and client code, but we still need to implement
and call the new method in the human-written parts of our example application.

### Update the server

In the same directory, open `bin/server.dart`. Implement the new method like
this:

```dart
class GreeterService extends GreeterServiceBase {
  @override
  Future<HelloReply> sayHello(ServiceCall call, HelloRequest request) async {
    return new HelloReply()..message = 'Hello, ${request.name}!';
  }

  @override
  Future<HelloReply> sayHelloAgain(
      ServiceCall call, HelloRequest request) async {
    return new HelloReply()..message = 'Hello again, ${request.name}!';
  }
}
...
```

### Update the client

In the same directory, open `bin/client.dart`. Call the new method like this:

```dart
Future<Null> main(List<String> args) async {
  final channel = new ClientChannel('localhost',
      port: 50051,
      options: const ChannelOptions(
          credentials: const ChannelCredentials.insecure()));
  final stub = new GreeterClient(channel);

  final name = args.isNotEmpty ? args[0] : 'world';

  try {
    var response = await stub.sayHello(new HelloRequest()..name = name);
    print('Greeter client received: ${response.message}');
    response = await stub.sayHelloAgain(new HelloRequest()..name = name);
    print('Greeter client received: ${response.message}');
  } catch (e) {
    print('Caught error: $e');
  }
  await channel.shutdown();
}
```

### Run!

Just like we did before, from the `example/helloworld` directory:

1. Run the server

   ```sh
   $ dart bin/server.dart
   ```

2. In another terminal, run the client

   ```sh
   $ dart bin/client.dart
   ```

## What's next

- Read a full explanation of how gRPC works in [What is gRPC?](../guides/)
  and [gRPC Concepts](../guides/concepts.html)
- Work through a more detailed tutorial in [gRPC Basics: Dart][]

[gRPC Basics: Dart]:../tutorials/basic/dart.html


## Reporting issues
Should you encounter an issue, please help us out by
<a href="https://github.com/grpc/grpc-dart/issues/new">filing issues</a>
in our issue tracker.</p>
