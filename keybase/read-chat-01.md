## Read Chat v.01

Initial test flow, reading a Keybase chat.

In this example, Keybase and NodeRED both running on an Ubuntu 18.04 web server. Keybase must be installed and available globally from the command line.

```json
[
    {
        "id": "ad92e293.bb1d28",
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
        "x": 120,
        "y": 60,
        "wires": [
            [
                "1f8408af.e608af"
            ]
        ]
    },
    {
        "id": "1f8408af.e608af",
        "type": "exec",
        "z": "63d957d.3a442a8",
        "command": "keybase chat read YOURNAME",
        "addpay": false,
        "append": "",
        "useSpawn": "false",
        "timer": "",
        "oldrc": false,
        "name": "",
        "x": 350,
        "y": 60,
        "wires": [
            [
                "a69132be.c7dc5"
            ],
            [],
            []
        ]
    },
    {
        "id": "a69132be.c7dc5",
        "type": "function",
        "z": "63d957d.3a442a8",
        "name": "",
        "func": "const chatArray = msg.payload.split(\"\\n[\");\narrayReformat(chatArray,outputArray);\n\n// tidy up each element, and output the array\nfunction arrayReformat(array, callback) {\n  chatArray.forEach(elementReformat);\n  callback();\n}\n\n// reformat elements, removing multiple spaces and new-lines\nfunction elementReformat(item,index) {\n  chatArray[index] = \"[\"+item;\n  chatArray[index] = chatArray[index].replace(/[\\n][\\s]{1,24}/g,\"\");\n  chatArray[index] = chatArray[index].replace(/[\\s]+$/,\"\");\n}\n\n// output the re-formatted array as 'payload'\nfunction outputArray() {\n  node.send({\"payload\":chatArray});\n  return;\n}",
        "outputs": 1,
        "noerr": 0,
        "x": 570,
        "y": 60,
        "wires": [
            [
                "c3aa0d4e.af64f"
            ]
        ]
    },
    {
        "id": "c3aa0d4e.af64f",
        "type": "debug",
        "z": "63d957d.3a442a8",
        "name": "",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "targetType": "msg",
        "x": 730,
        "y": 60,
        "wires": []
    }
]
```
