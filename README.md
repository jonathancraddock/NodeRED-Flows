# NodeRED-Flows

Backups of some NodeRED flows and scripts.

### Keybase

* [Read-Chat-01](./keybase/read-chat-01.md)  
* [Send-Chat-01](./keybase/send-chat-01.md)  

Examples above of simple Keybase read/send flows using the CLI only.

### Contents

* [BlythTide](./blythtide/readme.md)  *testing*  
* [Keybase Read Chat](./keybase/read-chat-01.md)  
* [Keybase Send Chat](./keybase/send-chat-01.md)  
* [Twitter 'Toot-Tweet'](./twitter/toot-tweet.md)  *in progress*  

## Require a NodeJS Library

If you've installed a library via NPM and want to make use of it in a Node-RED 'function' node, there are a couple of steps:

In `settings.js` modify/add the library. In this example, jsonwebtoken.

```js
functionGlobalContext: {
  //osModule:require('os'),
  jsonwebtoken:require('jsonwebtoken'),
}
```

Node-RED will require a restart at this point. Probably:
```bash
sudo systemctl restart node-red
```

In the function node, note that you can't simply use `const jwt = require('jsonwebtoken');`, but you can use `global.get` instead. Following the above example:
```js
// Load jsonwebtoken library
// 
const jwt = global.get('jsonwebtoken');

// Injected timestamp
let timeNow = msg.payload;

// Generate the JWT token
const payload = { 
    "sub": "What to do.",
    "name": "Vote for Pedro!!",
    "iat": timeNow
};
const secret = 'trust_NO_1';
const token = jwt.sign(payload, secret);

// Pass the generated token
msg.payload = token;

return msg;
```

This example makes the jsonwebtoken library available within the Node-RED function.

-----

# NodeRED with an Apache Reverse Proxy

> *These notes are from November 2019.*

There are plenty of guides for NodeRED and Nginx, but if you want to put NodeRED behind an Apache reverse proxy it's much harder to find any easy to follow guides. I can confirm the following config certainly works, but with a small disclaimer that it probably does require some further fine-tuning.

There were a few very helpful guides that don't mention *web sockets*. And, a couple that do remember to include that info don't remind you to enable proxy_wstunnel. Yes... it sounds obvious now, but I didn't find the errors very informative! ;-)

## Apache Modules

After installing Apache, I enabled the following modules:

```bash
sudo a2enmod ...
```

* proxy
* proxy_http
* proxy_balancer
* lbmethod_byrequests
* **proxy_wstunnel**
* xml2enc
* headers
* ssl

## Apache Reverse Proxy Config

The conf below is working ok. I'd like to enable TLSv1.3, but at he time of writing the Apache2 version in Ubuntu 18.04 LTS doesn't support it. Also worth noting that the choice of cipher suites below may prevent some out-of-date browsers from accessing the page.

```bash
<IfModule mod_ssl.c>
<VirtualHost *:443>

ServerAdmin webmaster@localhost
ServerName nodered.example.com

        SSLEngine on
        SSLHonorCipherOrder on
        SSLCompression off

        SSLOpenSSLConfCmd Protocol "-ALL, TLSv1.2"
        SSLOpenSSLConfCmd DHParameters "/etc/ssl/certs/dhparam.pem"

        SSLProtocol -all +TLSv1.2 
        # +TLSv1.3
        
        SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire

        SSLCipherSuite ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384

        Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
        Header always set X-Frame-Options SAMEORIGIN
        Header always set X-Content-Type-Options nosniff

        ProxyPreserveHost On

        ProxyPass               /comms        ws://localhost:1880/comms
        ProxyPassReverse        /comms        ws://localhost:1880/comms
        ProxyPass               /             http://127.0.0.1:1880/
        ProxyPassReverse        /             http://127.0.0.1:1880/

        SSLCertificateFile /etc/letsencrypt/live/nodered.example.com/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/nodered.example.com/privkey.pem
        Include /etc/letsencrypt/options-ssl-apache.conf

</VirtualHost>
</IfModule>
```

Also worth noting that the `Include /etc/letsencrypt/options-ssl-apache.conf` (which is added automatically by Certbot) modifies the SSL protocols and other settings.

```bash
# Intermediate configuration, tweak to your needs

#SSLProtocol             all -SSLv2 -SSLv3
#SSLCipherSuite          ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1$
#SSLHonorCipherOrder     on
#SSLCompression          off
```

I chose to comment out the lines in their *intermediate configuration* section as I had already set my own preferences and didn't want them being messed about.

A final note, SSL Stapling cannot be added in the virtual host conf and should be added to the `...mods-available/ssl.conf` file.

```bash
SSLUseStapling On
SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"
```
