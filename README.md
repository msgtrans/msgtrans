## MsgTrans
Description for MsgTrans (Message transmission framework) protocol.

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

import hunt.logging;
import hunt.util.serialize;

void main()
{
    // Create a service object alias "test"
    MessageTransportServer server = new MessageTransportServer("test");

    server.addChannel(new TcpServerChannel(9001));
    server.addChannel(new WebSocketServerChannel(9002, "/test"));

    server.acceptor((TransportContext ctx) {
        infof("New connection: id=%d", ctx.id());
    });

    server.start();
}

// Mark this executor as a "test" service
@TransportServer("test")
class MyExecutor : AbstractExecutor!(MyExecutor)
    @MessageId(MESSAGE.HELLO)
    void hello(TransportContext ctx, MessageBuffer buffer)
    {

        HelloMessage message = unserialize!HelloMessage(cast(const byte[])buffer.data);

        WelcomeMessage welcomeMessage = new WelcomeMessage;
        welcomeMessage.welcome = "Hello " ~ message.name;

        ctx.session().send(new MessageBuffer(MESSAGE.WELCOME, cast(ubyte[])serialize(welcomeMessage)));
    }
}
```

## Example for client

```D
import msgtrans;

import app.message.defined;
import app.message.HelloMessage;
import app.message.WelcomeMessage;

import hunt.logging;
import hunt.util.Serialize;

void main()
{
    // Create a client object alias "test"
    MessageTransportClient client = new MessageTransportClient("test");

    client.channel(new TcpClientChannel("127.0.0.1", 9001)).connect();

    auto message = new HelloMessage;
    message.name = "zoujiaqing";

    auto buffer = new MessageBuffer;
    buffer.id = MESSAGE.HELLO;
    buffer.data = cast(ubyte[]) serialize(message);

    client.send(buffer);

    client.block();
}

// Mark this executor for "test" client
@TransportClient("test")
class MyExecutor : AbstractExecutor!(MyExecutor)
{
    @MessageId(MESSAGE.WELCOME)
    void welcome(TransportContext ctx, MessageBuffer buffer)
    {
        auto message = unserialize!WelcomeMessage(cast(byte[]) buffer.data);

        infof("message: %s", message.welcome);
    }
}
```
