---
layout: post
title:  "golang websocket 举例"
date:   2019-03-07 23:27:02 +0800
comments: true
tags:
- golang
- websocket
---

### golang

```
package main

import (
	"log"
	"net/http"
	"golang.org/x/net/websocket"
)

func websocketLearning(ws *websocket.Conn){
	for{
		var reply string

		if errRecv := websocket.Message.Receive(ws, &reply);errRecv != nil{
			log.Println(errRecv)
			return
		}

		if errSend := websocket.Message.Send(ws, reply);errSend != nil{
			log.Println(errSend)
			log.Println("Client connect error")
			return
		}
	}

}

func main(){
	http.Handle("/", websocket.Handler(websocketLearning))

	if err := http.ListenAndServe(":1235", nil); err != nil {
		log.Fatal("ListenAndServe:", err)
	}
}
```

### js
代码包含其他内容， 不过修改端口和URL即可运行

```
<html>
<head></head>
<body>
<script type="text/javascript">
	var sock = null;
	var wsuri = "ws://127.0.0.1:1234/web_socket";
    var heart_beat = 0;
    var need_heartbeat = false;

	window.onload = conn_server();

	function send() {
		var msg = document.getElementById('message').value;
		if (sock.readyState == 1){
			sock.send(msg);
        }
        else{
        	alert("Connection Error")
        }

	}

    function show(){
    	if (need_heartbeat === true)
        {
            heart_beat = heart_beat + 1;
            console.log("Heartbeat send", heart_beat);
            if (sock.readyState !== 1){
                console.log("Connection Error");
                if (document.getElementById('conn_number').innerHTML==="Online"){
                    document.getElementById('conn_number').innerHTML="Offline";
                }
            }
            else{
            	sock.send("2018");
            }
        }
    }

    function conn_server(){
			console.log("onload");
			sock = new WebSocket(wsuri);
			sock.onopen = function() {
				document.getElementById('conn_number').innerHTML="Online";
				console.log("connected to " + wsuri);
                need_heartbeat = true;
				setInterval(show, 3000);
			};
			sock.onclose = function(e) {
				document.getElementById('conn_number').innerHTML="Offline";
				console.log("connection closed (" + e.code + ")");
                need_heartbeat = false;
			};
			sock.onmessage = function(e) {
				console.log("message received: " + e.data);
				if (e.data.startsWith('ConnNumber:')){
					document.getElementById('conn_number').innerHTML=e.data.split(":")[1];
				}
				else {
					document.getElementById('recv_msg').innerHTML=e.data;
				}
			}
		}
</script>
<h1>WebSocket Echo Test</h1>
<form>
    <p>
        Status:<span id="conn_number"></span>
    </p>
    <p>
        Message: <input id="message" type="text" value="Hello, world!">
    </p>
</form>
<button onclick="send();">Send Message</button>
<button onclick="conn_server();">Re-build connetcion</button>
<p>
Recv: <span id="recv_msg"></span>
</body>
</html>
```
