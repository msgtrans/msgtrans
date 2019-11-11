## MsgTrans
Description for MsgTrans protocol.

## Usage
MsgTrans uses TCP / Websockt / UDP( QUIC or Aeron ) as the transport layer.

## Example for message define

HelloMessage code:
```D
module app.message.HelloMessage;

class HelloMessage
{
    string name;
}
```

WelcomeMessage code:
```D
module app.message.WelcomeMessage;

class WelcomeMessaage
{
    string welcome;
}
```

Defined messages:
```D
module app.message.defined;

enum MESSAGE {
    HELLO = 10001,
    WELCOME = 20001
}
```

## Example for server

```D
import msgtrans;

import app.message.defined;
import app.message.HelloMessage;
import app.message.WelcomeMessage;

import std.datetime : seconds;

class MyExecutor : MessageExecutor
{
    @MessageId(MESSAGE.HELLO)
    void hello(Context ctx, ubyte[] data)
    {
        string msg = cast(string) data;

        string welcome = "Welcome " ~ msg;

        ctx.send(MESSAGE.WELCOME, welcome.dup);
    }
}

void main()
{
    auto server = MessageTransportServer;

    server.addTransport(new TcpTransport(9001));
    server.addTransport(new WebsocketTransport(9002, "/test"));

    server.addExecutor(MyExecutor);

    server.codec(new CustomCodec).keepAliveAckTimeout(60.seconds).start().block();
}
```

## Example for client

```D
import msgtrans;

import app.message.defined;
import app.message.HelloMessage;
import app.message.WelcomeMessage;

class MyExecutor : MessageExecutor
{
    @MessageId(MESSAGE.WELCOME)
    void welcome(Context ctx, Object msg)
    {
        auto message = cast(WelcomeMessage) msg;

        import std.stdio : writeln;

        writeln(message.welcome);
    }
}

void main()
{
    auto client = MessageTransportClient;

    client.transport(new WebsocketTransport("ws://msgtrans.huntlabs.net:9002/test"));

    client.addExecutor(new MyExecutor);

    client.codec(new CustomCodec).keepAlive().connect();

    auto message = new HelloMessage;
    message.name = "zoujiaqing";

    client.send(MESSAGE.HELLO, message);

    client.block();
}
```
