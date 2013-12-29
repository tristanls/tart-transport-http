# tart-transport-http

_Stability: 1 - [Experimental](https://github.com/tristanls/stability-index#stability-1---experimental)_

[![NPM version](https://badge.fury.io/js/tart-transport-http.png)](http://npmjs.org/package/tart-transport-http)

HTTP transport implementation for [Tiny Actor Run-Time in JavaScript](https://github.com/organix/tartjs).

## Contributors

[@dalnefre](https://github.com/dalnefre), [@tristanls](https://github.com/tristanls)

## Overview

An implementation of a HTTP transport for [Tiny Actor Run-Time in JavaScript](https://github.com/organix/tartjs).

  * [Usage](#usage)
  * [Tests](#tests)
  * [Documentation](#documentation)
  * [Sources](#sources)

## Usage

To run the below example run:

    npm run readme

```javascript
"use strict";

var http = require('http');
var tart = require('tart');
var transport = require('../index.js');

var sponsor = tart.minimal();

var send = sponsor(transport.sendBeh);

var receivedMessageCount = 0;
var receptionist = sponsor(function (message) {
    console.log('received message:', message);
    receivedMessageCount++;
    if (receivedMessageCount >= 2) {
        close(); // close listening server
    }
});

var serverCapabilities = transport.server(receptionist);
var close = sponsor(serverCapabilities.closeBeh);
var listen = sponsor(serverCapabilities.listenBeh);

var fail = sponsor(function (error) {
    console.dir(error);
});

var listenAck = sponsor(function listenAckBeh(message) {
    console.log('transport listening on http://' + message.host + ':' + message.port);
    send({
        address: 'http://localhost:7847/#t5YM5nxnJ/xkPTo3gtHEyLdwMRFIwyJOv5kvcFs+FoMGdyoDNgSLolq0',
        content: '{"some":{"json":"content"},"foo":true}',
        fail: fail,
        ok: function () {
            console.log('foo sent');
        }
    });
    send({
        address: 'http://localhost:7847/#I0InGCVn0ApX0YBnF5+JFMheKOajHkaTrNthYRI2hOj4GrM5IaWO1Cv0',
        content: '{"some":{"json":"content"},"bar":true}',
        fail: fail,
        ok: function () {
            console.log('bar sent');
        }
    });    
});

listen({
    host: 'localhost', 
    port: 7847, 
    ok: listenAck,
    fail: fail
});
```

## Tests

    npm test

## Documentation

**Public API**

  * [transport.sendBeh](#transportsendbeh)
  * [transport.server(receptionist)](#transportserverreceptionist)
  * [serverCapabilities.closeBeh](#servercapabilitiesclosebeh)
  * [serverCapabilities.listenBeh](#servercapabilitieslistenbeh)

### transport.sendBeh

Actor behavior that will attempt to send messages over TLS.

Message format:

  * `address`: _String_ HTTP address in URI format. Scheme, host, and port are required. Framgment is optional but usually necessary. For example: `http://localhost:7847/#t5YM5nxnJ/xkPTo...`. 
  * `content`: _String_ JSON content to be sent.
  * `fail`: _Actor_ `function (error) {}` _(Default: undefined)_ Optional actor to report `error` (if any).
  * `ok`: _Actor_ `function () {}` _(Default: undefined)_ Optional actor to report successful send to the destination.

```javascript
var send = sponsor(transport.sendBeh);
send({
    address: 'http://localhost:7847/#ZkiLrAwGX7N1eeOXXMAeoVp7vsYJKeISjfT5fESfkRiZOIpkPx1bAS8y', 
    content: '{"some":{"json":"content"}}'
});
```

### transport.server(receptionist)

  * `receptionist`: _Actor_ `function (message) {}` Actor to forward traffic received by this server in `{address: <URI>, contents: <json>}` format.
  * Return: _Object_ An object containing behaviors for listen and close capabilities.
    * `closeBeh`: [serverCapabilities.closeBeh](#servercapabilitiesclosebeh)
    * `listenBeh`: [serverCapabilities.listenBeh](#servercapabilitieslistenbeh)

Creates an entangled pair of capabilities that will control a single HTTP server.

### serverCapabilities.closeBeh

Actor behavior that will close a listening server.

Message is an `ack` _Actor_ `function () {}`, an actor that will be sent an empty message once the server closes.

```javascript
var serverCapabilities = transport.server(receptionist);
var close = sponsor(serverCapabilities.closeBeh);
close(sponsor(function ack() {
    console.log('acked close'); 
});
```

### serverCapabilities.listenBeh

Actor behavior that will create a new listening HTTP server.

Message format:

  * `host`: _String_ HTTP host to listen on.
  * `port`: _Number_ HTTP port to listen on.
  * `ok`: _Actor_ `function (message) {}` Optional actor to receive acknowledgment once the server is listening.
  * `fail`: _Actor_ `function (error) {}` Optional actor to receive any errors when starting the HTTP transport.

```javascript
var serverCapabilities = transport.server(receptionist);
var listen = sponsor(serverCapabilities.listenBeh);
listen({
    host: 'localhost',
    port: 7847,
    ok: sponsor(function listenAckBeh(message) {
        console.log('transport listening on http://' + message.host + ':' + message.port);
    }),
    fail: sponsor(function failBeh(message) {
        console.error(message);
    })
});
```

## Sources

  * [Tiny Actor Run-Time (JavaScript)](https://github.com/organix/tartjs)