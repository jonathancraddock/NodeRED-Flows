## Read Chat v.01

Initial test flow, reading a Keybase chat.

![](https://github.com/jonathancraddock/NodeRED-Flows/raw/master/keybase/img/keybase-chat-read-01.PNG "Read 01")

In this example, Keybase and NodeRED both running on an Ubuntu 18.04 web server. Keybase must be installed and available globally from the command line.

There may be a simpler way to achieve this, but I found it's not sufficient to split the array based on line-feeds, since longer messages will include multiple line-feeds and multiple blocks of leading spaces. The `elementReformat` function replaces the `[` that was lost in the initial split, and subsequently gets rid of line-feeds that are followed by 24 spaces. (This merges long messages into one string.) I also found there are sometimes multiple trailing spaces, and these are removed.

The structure of the node meant that `node.send` is required; the array is sent as the payload.

```javascript
const chatArray = msg.payload.split("\n[");
arrayReformat(chatArray,outputArray);

// tidy up each element, and output the array
function arrayReformat(array, callback) {
  chatArray.forEach(elementReformat);
  callback();
}

// reformat elements, removing multiple spaces and new-lines
function elementReformat(item,index) {
  chatArray[index] = "["+item;
  chatArray[index] = chatArray[index].replace(/[\n][\s]{1,24}/g,"");
  chatArray[index] = chatArray[index].replace(/[\s]+$/,"");
}

// output the re-formatted array as 'payload'
function outputArray() {
  node.send({"payload":chatArray});
  return;
}
```

Copy of NodeRED flow below.

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
