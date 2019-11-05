## MsgTrans
description for MsgTrans protocol.

## Usage
MsgTrans uses TCP / Websockt / UDP as the transport layer.

## Example for message define

HelloMessage code:
```D
module app.message.HelloMessage;

class HelloMessage
{
    int messageId = 10001;

    string name;
}
```

WelcomeMessage code:
```D
module app.message.WelcomeMessage;

class WelcomeMessaage
{
    int messageId = 20001;

    string welcome;
}
```

## Example for server

```D
import msgtrans;

import app.message.HelloMessage;
import app.message.WelcomeMessage;

import std.datetime : seconds;

class MyExecutor : MessageExecutor
{
    @MessageId(10001)
    void hello(Context ctx, Object msg)
    {
        auto helloMessage = cast(HelloMessage) msg;

        auto welcomeMessage = new WelcomeMessage;
        welcomeMessage.welcome = "Welcome " ~ helloMessage.name;

        ctx.send(welcomeMessage);
    }
}

void main()
{
	auto server = MessageTransportServer;

	server.addTransport(new TcpTransport(9001));
	server.addTransport(new WebsocketProtocol("ws://localhost:9002/test"));

    server.addService(MyService);

    server.codec(new ProtobufCodec).keepAliveAckTimeout(60.seconds).start().block();
}
```

## Example for client

```D
import msgtrans;

import app.message.HelloMessage;
import app.message.WelcomeMessage;

import std.stdio : writeln;
import std.datetime : seconds;

import core.thread : sleep;

class MyExecutor : MessageExecutor
{
    @MessageId(20001)
    void welcome(Context ctx, Object msg)
    {
        auto message = cast(WelcomeMessage) msg;

        writeln(message.welcome);
    }
}

void main()
{
	auto client = MessageTransportClient;

	client.transport(new WebsocketTransport("ws://msgtrans.huntlabs.net:9002/test")).connect().codec(new ProtobufCodec).keepAlive();
    
    client.addExecutor(new MyExecutor);

    auto message = new HelloMessage;
    message.name = "zoujiaqing";

	client.send(message);

    while (client.alive())
    {
        Thread.sleep(5.seconds);
    }
}
```
