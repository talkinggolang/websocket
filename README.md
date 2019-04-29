# websocket

[toc]

## 简介
WebSocket是一种在单个TCP连接上进行全双工通信的协议,WebSocket通信协议于2011年被IETF定为标准RFC 6455,WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输，就像在使用一个常规的TCP Socket一样。它解决了Web实时化的问题，相比传统HTTP有如下好处：

- 一个Web客户端只建立一个TCP连接
 - WebSocket服务端可以推送(push)数据到web客户端.
- 有更加轻量级的头，减少数据传送量

## WebSocket & HTTP

WebSocket URL的起始输入是ws://或是wss://（在SSL上）。
WebSocket是一种与HTTP不同的协议。这两个协议都位于OSI模型的第7层，并依赖于第4层的TCP。虽然它们不同，但RFC 6455规定WebSocket“设计为通过HTTP端口80和443工作，以及支持HTTP代理和中介”从而使其与HTTP协议兼容。为了实现兼容性，WebSocket握手使用HTTP Upgrade头从HTTP协议更改为WebSocket协议。

## 原理
WebSocket的协议颇为简单，在第一次handshake通过以后，连接便建立成功，其后的通讯数据都是以”\x00″开头，以”\xFF”结尾。在客户端，这个是透明的，WebSocket组件会自动将原始数据“掐头去尾”。

浏览器发出WebSocket连接请求，然后服务器发出回应，然后连接建立成功，这个过程通常称为“握手” (handshaking)。请看下面的请求和反馈信息：

handshaking request(HTTP):
```plain
GET /echo HTTP/1.1
Host: 127.0.0.1:8000
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: KbSSEFTG2HqtRGcKXRJ/Lw==
Origin: http://127.0.0.1:8000/
Sec-WebSocket-Version: 13
```
handshaking response:
```plain
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: fFBooB7FAkLlXgRSz0BT3v4hq5s=
Sec-WebSocket-Location: ws://example.com/
```

- Connection必须设置Upgrade，表示客户端希望连接升级。
- Upgrade字段必须设置Websocket，表示希望升级到Websocket协议。
- Sec-WebSocket-Key是随机的字符串，服务器端会用这些数据来构造出一个SHA-1的信息摘要。把“Sec-WebSocket-Key”加上一个特殊字符串“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”，然后计算SHA-1摘要，之后进行BASE-64编码，将结果做为“Sec-WebSocket-Accept”头的值，返回给客户端。如此操作，可以尽量避免普通HTTP请求被误认为Websocket协议。
- Sec-WebSocket-Version 表示支持的Websocket版本。RFC6455要求使用的版本是13，之前草案的版本均应当弃用。
- Origin字段是可选的，通常用来表示在浏览器中发起此Websocket连接所在的页面，类似于Referer。但是，与Referer不同的是，Origin只包含了协议和主机名称。
- 其他一些定义在HTTP协议中的字段，如Cookie等，也可以在Websocket中使用。
[^1]
## 开发包选择

 go 标准SDK 没有对ws的支持 `x/net/websocket` 和 `gorilla/websocket` 选择

### Gorilla WebSocket compared with other packages

<table>
<tr>
<th></th>
<th><a href="http://godoc.org/github.com/gorilla/websocket">github.com/gorilla</a></th>
<th><a href="http://godoc.org/golang.org/x/net/websocket">golang.org/x/net</a></th>
</tr>
<tr>
<tr><td colspan="3"><a href="http://tools.ietf.org/html/rfc6455">RFC 6455</a> Features</td></tr>
<tr><td>Passes <a href="http://autobahn.ws/testsuite/">Autobahn Test Suite</a></td><td><a href="https://github.com/gorilla/websocket/tree/master/examples/autobahn">Yes</a></td><td>No</td></tr>
<tr><td>Receive <a href="https://tools.ietf.org/html/rfc6455#section-5.4">fragmented</a> message<td>Yes</td><td><a href="https://code.google.com/p/go/issues/detail?id=7632">No</a>, see note 1</td></tr>
<tr><td>Send <a href="https://tools.ietf.org/html/rfc6455#section-5.5.1">close</a> message</td><td><a href="http://godoc.org/github.com/gorilla/websocket#hdr-Control_Messages">Yes</a></td><td><a href="https://code.google.com/p/go/issues/detail?id=4588">No</a></td></tr>
<tr><td>Send <a href="https://tools.ietf.org/html/rfc6455#section-5.5.2">pings</a> and receive <a href="https://tools.ietf.org/html/rfc6455#section-5.5.3">pongs</a></td><td><a href="http://godoc.org/github.com/gorilla/websocket#hdr-Control_Messages">Yes</a></td><td>No</td></tr>
<tr><td>Get the <a href="https://tools.ietf.org/html/rfc6455#section-5.6">type</a> of a received data message</td><td>Yes</td><td>Yes, see note 2</td></tr>
<tr><td colspan="3">Other Features</tr></td>
<tr><td><a href="https://tools.ietf.org/html/rfc7692">Compression Extensions</a></td><td>Experimental</td><td>No</td></tr>
<tr><td>Read message using io.Reader</td><td><a href="http://godoc.org/github.com/gorilla/websocket#Conn.NextReader">Yes</a></td><td>No, see note 3</td></tr>
<tr><td>Write message using io.WriteCloser</td><td><a href="http://godoc.org/github.com/gorilla/websocket#Conn.NextWriter">Yes</a></td><td>No, see note 3</td></tr>
</table>

Notes:

1. Large messages are fragmented in [Chrome's new WebSocket implementation](http://www.ietf.org/mail-archive/web/hybi/current/msg10503.html).
2. The application can get the type of a received data message by implementing
   a [Codec marshal](http://godoc.org/golang.org/x/net/websocket#Codec.Marshal)
   function.
3. The go.net io.Reader and io.Writer operate across WebSocket frame boundaries.
  Read returns when the input buffer is full or a frame boundary is
  encountered. Each call to Write sends a single frame message. The Gorilla
  io.Reader and io.WriteCloser operate on a single WebSocket message.[^2]


## echo服务器

### `x/net/websocket`

- server

``` golang
package main

import (
	"fmt"
	"net/http"

	"golang.org/x/net/websocket"
)

func main() {
	http.Handle("/echo", websocket.Handler(handler))
	http.ListenAndServe(":6666", nil)

}

func handler(con *websocket.Conn) {

    defer con.Close()
	for {
		tmp := make([]byte, 128)
		readLen, err := con.Read(tmp)
		if err != nil {
			fmt.Println(err)
			break
		}
		fmt.Println("recive", readLen, "bytes")
        if readLen == 0 {
			break
		} else {
			fmt.Println(string(tmp))
			con.Write([]byte(tmp))
		}
	}
}
```

- client

``` go
package main

import (
	"fmt"

	"golang.org/x/net/websocket"
)

func main() {
	conn, err := websocket.Dial("ws://127.0.0.1:6666/echo", "", "http://127.0.0.1:6666/")
	if err != nil {
		panic(err)
	}

	go func() {
		for {
			tmp := make([]byte, 128)
			l, err := conn.Read(tmp)
			if err != nil {
				fmt.Println("read failed ", err)
				break
			}
			if l == 0 {
				fmt.Println("conn close")
				break
			} else {
				fmt.Println("echo", string(tmp))
			}
		}
	}()

	for {
		var input string
		fmt.Scanln(&input)
		conn.Write([]byte(input))
	}
}

```

### `gorilla/websocket`
- server

```go
package main

import (
	"log"
	"net/http"

	"github.com/gorilla/websocket"
)

func main() {
	http.HandleFunc("/echo", echo)
	http.ListenAndServe("localhost:8080", nil)
}

var upgrader = websocket.Upgrader{} // use default options

func echo(w http.ResponseWriter, r *http.Request) {
	c, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Print("upgrade:", err)
		return
	}
	defer c.Close()
	for {
		mt, message, err := c.ReadMessage()
		if err != nil {
			log.Println("read:", err)
			break
		}
		log.Printf("recv: %s", message)
		err = c.WriteMessage(mt, message)
		if err != nil {
			log.Println("write:", err)
			break
		}
	}
}

```


- client
``` golang
package main

import (
	"log"
	"net/http"

	"github.com/gorilla/websocket"
)

func main() {
	http.HandleFunc("/echo", echo)
	http.ListenAndServe("localhost:8080", nil)
}

var upgrader = websocket.Upgrader{} // use default options

func echo(w http.ResponseWriter, r *http.Request) {
	c, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Print("upgrade:", err)
		return
	}
	defer c.Close()
	for {
		mt, message, err := c.ReadMessage()
		if err != nil {
			log.Println("read:", err)
			break
		}
		log.Printf("recv: %s", message)
		err = c.WriteMessage(mt, message)
		if err != nil {
			log.Println("write:", err)
			break
		}
	}
}

```

### 一个跟加高级的封装 melody

Melody is websocket framework based on [github.com/gorilla/websocket](https://github.com/gorilla/websocket)
that abstracts away the tedious parts of handling websockets. It gets out of
your way so you can write real-time apps. Features include:

* [x] Clear and easy interface similar to `net/http` or Gin.
* [x] A simple way to broadcast to all or selected connected sessions.
* [x] Message buffers making concurrent writing safe.
* [x] Automatic handling of ping/pong and session timeouts.
* [x] Store data on sessions.

1. HandleRequest 升级成ws链接
2. HandleConnect
3. HandleDisconnect
4. HandleMessage
5. HandleSentMessage
6. Broadcast
7. BroadcastFilter

图 melody-connction

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
	"gopkg.in/olahol/melody.v1"
)

func main() {
	r := gin.Default()
	m := melody.New()

	r.GET("/", func(c *gin.Context) {
		http.ServeFile(c.Writer, c.Request, "index.html")
	})

	//发送握手协议
	r.GET("/ws", func(c *gin.Context) {
		m.HandleRequest(c.Writer, c.Request)
	})

	m.HandleMessage(func(s *melody.Session, msg []byte) {
		m.Broadcast(msg)
		m.BroadcastFilter(msg, func(s *melody.Session) bool {
			return false
		})
	})

	m.HandleSentMessage(func(s *melody.Session, msg []byte) {

		fmt.Println(m.CloseWithMsg([]byte("byebye")))
	})

	r.Run(":5000")
}

```

## gRPC

### gRPC的四种模式
``` protobuf
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

1. Unary RPCs where the client sends a single request to the server and gets a single response back, just like a normal function call.
``` protobuf
rpc SayHello(HelloRequest) returns (HelloResponse){
}
```
2. Server streaming RPCs where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. gRPC guarantees message ordering within an individual RPC call.
```protobuf
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse){
}
```
3. Client streaming RPCs where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them and return its response. Again gRPC guarantees message ordering within an individual RPC call.
``` protobuf
rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {
}
```
4. Bidirectional streaming RPCs where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved.
``` protobuf
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){
}
```
[^3]

### grpc web
![Alt text](./grpcweb.png)


### WebSocket and gRPC

#### melody + grpc

用 melody 做一层 WebSoket的网关，后端使用gRPC 协议做业务逻辑
```plain
                         +-------------+
                         |   browser   |
                         |             |
                         +-------------+
                                |
                                |
                                |
                                |
     +--------------+    +------v-------+
     | gRPC server  |    |    ws server |
     |              +<---+              |
     +--------------+    +--------------+

```

对应的包类型 嵌套包
``` plain

    +-----------------------------+
    | 4 | 4 | 4 |                 |
    |   |   |   |      content    |
    +-----------------------------+
      |    |   |
      v    |   v
   msg type|  who
           |
           v
         msg len

```
调用图

![Alt text](./melody.png)

### WebSocket + gRPC

#### gRPC gateway

为gRPC 服务提供 HTTP + JSON 的接口 
![Alt text](./grpc-rest-gateway.png)


#### gRPC WebSocket proxy

在gRPC gateway 的基础上，将连接升级成ws
![Alt text](./grpc_ws_proxy.png)

精简代码实现
```go
func (p *Proxy) proxy(w http.ResponseWriter, r *http.Request) {

	var responseHeader http.Header
	// If Sec-WebSocket-Protocol starts with "Bearer", respond in kind.
	// TODO(tmc): consider customizability/extension point here.
	if strings.HasPrefix(r.Header.Get("Sec-WebSocket-Protocol"), "Bearer") {
		responseHeader = http.Header{
			"Sec-WebSocket-Protocol": []string{"Bearer"},
		}
	}

	conn, err := upgrader.Upgrade(w, r, responseHeader)
	if err != nil {
		p.logger.Warnln("error upgrading websocket:", err)
		return
	}
	defer conn.Close()

	requestBodyR, requestBodyW := io.Pipe()
	request, err := http.NewRequest(r.Method, r.URL.String(), requestBodyR)
	if err != nil {
		p.logger.Warnln("error preparing request:", err)
		return
	}
	if swsp := r.Header.Get("Sec-WebSocket-Protocol"); swsp != "" {
		request.Header.Set("Authorization", transformSubProtocolHeader(swsp))
	}

	if p.requestMutator != nil {
		request = p.requestMutator(r, request)
	}

	responseBodyR, responseBodyW := io.Pipe()
	response := newInMemoryResponseWriter(responseBodyW)

	go func() {
		defer cancelFn()
		p.h.ServeHTTP(response, request) // 此处去调用自己实现的handle
	}()

	// read loop -- take messages from websocket and write to http request
	go func() {
	
		for {
			_, payload, err := conn.ReadMessage()
			n, err := requestBodyW.Write(payload)
			requestBodyW.Write([]byte("\n"))
		}
	}()
	// write loop -- take messages from response and write to websocket
	scanner := bufio.NewScanner(responseBodyR)
	for scanner.Scan() {
		conn.WriteMessage(websocket.TextMessage, 
}
```


[^1]:https://zh.wikipedia.org/wiki/WebSocket

[^2]:https://raw.githubusercontent.com/gorilla/websocket/master/README.md

[^3]: https://grpc.io/docs/guides/concepts.html



