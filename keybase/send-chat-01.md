## Send Chat v.01

Send simple message via Keybase, using a NodeRED exec node.

![](https://raw.githubusercontent.com/jonathancraddock/NodeRED-Flows/master/keybase/img/keybase-chat-send-01.PNG "Send 01")

Set message as the payload, remember to escape the quotes.

```javascript
msg.payload = "\"Text of message goes here.\"";
return msg;
```

Flow copied below.

```json
[
    {
        "id": "a77734d6.63b408",
        "type": "inject",
        "z": "63d957d.3a442a8",
        "name": "",
        "topic": "",
        "payload": "",
        "payloadType": "date",
        "repeat": "",
        "crontab": "",
        "once": false,
        "onceDelay": 0.1,
        "x": 140,
        "y": 60,
        "wires": [
            [
                "e76ee27a.8ea03"
            ]
        ]
    },
    {
        "id": "44ace58.0cc211c",
        "type": "debug",
        "z": "63d957d.3a442a8",
        "name": "",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "false",
        "x": 750,
        "y": 60,
        "wires": []
    },
    {
        "id": "e76ee27a.8ea03",
        "type": "function",
        "z": "63d957d.3a442a8",
        "name": "",
        "func": "msg.payload = \"\\\"Text of message goes here.\\\"\";\n\nreturn msg;",
        "outputs": 1,
        "noerr": 0,
        "x": 290,
        "y": 60,
        "wires": [
            [
                "f4577ace.dad9b8"
            ]
        ]
    },
    {
        "id": "f4577ace.dad9b8",
        "type": "exec",
        "z": "63d957d.3a442a8",
        "command": "keybase chat send YOURNAME ",
        "addpay": true,
        "append": "",
        "useSpawn": "false",
        "timer": "",
        "oldrc": false,
        "name": "",
        "x": 510,
        "y": 60,
        "wires": [
            [
                "44ace58.0cc211c"
            ],
            [],
            []
        ]
    }
]
```
