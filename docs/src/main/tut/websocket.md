---
layout: default
title: Websockets
---

## Websockets

Websockets are handled a bit differently than regular requests, they're not a
single action, rather a stream of incoming and outgoing data. So you'll have to
provide these two streams when creating the websocket. The two streams are
gathered in an `Exchange`.

```tut
import org.http4s._
import org.http4s.server.websocket.WS
import org.http4s.websocket.WebsocketBits._
import org.http4s.dsl._
import scalaz.stream.{Exchange, Process, Sink}
import scalaz.concurrent.Task

val textEchoService = HttpService {
  case GET -> Root / "websocket" => {
    val toClient = Process("Foo", "Bar").map(x => Text(x))
    val receiving: Sink[Task, String] = Process.constant(x => Task.delay(println(x)))
    val fromClient = receiving.contramap[WebSocketFrame]({
      case Text(t, _) => t
      case x => throw new IllegalArgumentException(s"Unexpected message: ${x}")
    })
    WS(Exchange(toClient, fromClient))
  }
}
```
