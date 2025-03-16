---
title: Debugging Node.js
layout: learn
---

# Debugging Node.js

* goal
  * debugging your Node.js apps & scripts

## Enable Inspector

* `--inspect`
  * Node.js process
    * -- listens (by default "127.0.0.1:9229") for a -- debugging client
    * -- assign an -- unique [UUID][]

* Inspector clients -- must -- know
  * host address,
  * port,
  * UUID
  ```
  # ws://hostAdress:port/UUID
  ws://127.0.0.1:9229/0f2c936f-b1cd-4ac9-aab3-f63b0f33d55e
  ```

* Node.js -- can listen for -- debugging messages
  * requirements
    * `SIGUSR1` signal
      * NOT available | Windows
  * |
    * Node.js v7-
      * -- activates the -- legacy Debugger API
    * Node.js v8+
      * -- activates the -- Inspector API

## Security Implications

* 👀if malicious actor -- connect to -- this port -> can execute arbitrary code -- on behalf of the -- Node.js process👀
  * Reason: 🧠 debugger -- has FULL access to the -- Node.js execution environment

### Exposing the debug port PUBLICLY

* scenario
  * debugger -- is bound to a --
    * public IP address
    * 0.0.0.0

* if clients -- can reach -- your IP address -> able to connect | debugger, WITHOUT restriction
  * == run arbitrary code

* recommendations
  * ensure firewalls & access controls

* see [Enabling remote debugging scenarios](#enabling-remote-debugging-scenarios)

### Local applications -- have -- FULL access to the inspector

* scenario
  * applications running locally & default inspector port (127.0.0.1)

### Browsers, WebSockets and same-origin policy

* TODO:
Websites open in a web-browser can make WebSocket and HTTP requests under the
browser security model. An initial HTTP connection is necessary to obtain a
unique debugger session id. The same-origin-policy prevents websites from being
able to make this HTTP connection. For additional security against
[DNS rebinding attacks](https://en.wikipedia.org/wiki/DNS_rebinding), Node.js
verifies that the 'Host' headers for the connection either
specify an IP address or `localhost` precisely.

These security policies disallow connecting to a remote debug server by
specifying the hostname. You can work-around this restriction by specifying
either the IP address or by using ssh tunnels as described below.

## Inspector Clients

* `node inspect myscript.js`
  * == minimal CLI debugger
  * -- can be connected by --
    * commercial tools
    * open source tools

### Chrome DevTools 55+, Microsoft Edge

* ways
  * **Option 1**
    * | Chromium-based browser, open `chrome://inspect` OR | Edge, open `edge://inspect`
    * Configure button, ensure your target host & port are listed
  * **Option 2**
    * copy
      * `/json/list`'s `devtoolsFrontendUrl` output OR
      * --inspect hint text
    * paste | Chrome

* see
  * https://github.com/ChromeDevTools/devtools-frontend,
  * https://www.microsoftedgeinsider.com

### Visual Studio Code 1.10+

* TODO:
- In the Debug panel, click the settings icon to open `.vscode/launch.json`.
  Select "Node.js" for initial setup.

See https://github.com/microsoft/vscode for more information.

### Visual Studio 2017+

- Choose "Debug > Start Debugging" from the menu or hit F5.
- [Detailed instructions](https://github.com/Microsoft/nodejstools/wiki/Debugging).

### JetBrains IDEs

* create a NEW Node.js debug configuration
* hit Debug
* `--inspect` -- used by default by -- Node.js 7+
  * if you want to disable -> uncheck `js.debugger.node.use.inspect` | IDE Registry

* see [WebStorm online help](https://www.jetbrains.com/help/webstorm/running-and-debugging-node-js.html)

### chrome-remote-interface

- Library to ease connections to [Inspector Protocol][] endpoints.

See https://github.com/cyrus-and/chrome-remote-interface for more information.

### Gitpod

- Start a Node.js debug configuration from the `Debug` view or hit `F5`. [Detailed instructions](https://medium.com/gitpod/debugging-node-js-applications-in-theia-76c94c76f0a1)

See https://www.gitpod.io for more information.

### Eclipse IDE with Eclipse Wild Web Developer extension

- From a .js file, choose "Debug As... > Node program", or
- Create a Debug Configuration to attach debugger to running Node.js application (already started with `--inspect`).

See https://eclipse.org/eclipseide for more information.

## Command-line options

* runtime flags -- about -- debugging

| Flag                               | Meaning                                                                                                                                               |
|------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| --inspect                          | Enable inspector agent <br/> Listen \| default address and port (127.0.0.1:9229)                                                                      |
| --inspect=[host:port]              | Enable inspector agent <br/> bind -- to -- address OR hostname host (default: 127.0.0.1) <br/> listen \| default port (default: 9229)                 |
| --inspect-brk                      | Enable inspector agent; Listen on default address and port (127.0.0.1:9229); Break before user code starts                                            |
| --inspect-brk=[host:port]          | Enable inspector agent; Bind to address or hostname host (default: 127.0.0.1); Listen on port port (default: 9229); Break before user code starts     |
| --inspect-wait                     | Enable inspector agent; Listen on default address and port (127.0.0.1:9229); Wait for debugger to be attached.                                        |
| --inspect-wait=[host:port]         | Enable inspector agent; Bind to address or hostname host (default: 127.0.0.1); Listen on port port (default: 9229); Wait for debugger to be attached. |
| node inspect script.js             | Spawn child process to run user's script under --inspect flag; and use main process to run CLI debugger.                                              |
| node inspect --port=xxxx script.js | Spawn child process to run user's script under --inspect flag; and use main process to run CLI debugger. Listen on port port (default: 9229)          |

## Enabling remote debugging scenarios

We recommend that you never have the debugger listen on a public IP address. If
you need to allow remote debugging connections we recommend the use of ssh
tunnels instead. We provide the following example for illustrative purposes only.
Please understand the security risk of allowing remote access to a privileged
service before proceeding.

Let's say you are running Node.js on a remote machine, remote.example.com, that
you want to be able to debug. On that machine, you should start the node process
with the inspector listening only to localhost (the default).

```bash
node --inspect server.js
```

Now, on your local machine from where you want to initiate a debug client
connection, you can setup an ssh tunnel:

```bash
ssh -L 9221:localhost:9229 user@remote.example.com
```

This starts a ssh tunnel session where a connection to port 9221 on your local
machine will be forwarded to port 9229 on remote.example.com. You can now attach
a debugger such as Chrome DevTools or Visual Studio Code to localhost:9221,
which should be able to debug as if the Node.js application was running locally.

## Legacy Debugger

* ⚠️deprecated -- from -- Node.js v7.7.0+ ⚠️
* recommendations
  * use `--inspect` & Inspector

* TODO:
When started with the **--debug** or **--debug-brk** switches in version 7 and
earlier, Node.js listens for debugging commands defined by the discontinued
V8 Debugging Protocol on a TCP port, by default `5858`. Any debugger client
which speaks this protocol can connect to and debug the running process; a
couple popular ones are listed below.

The V8 Debugging Protocol is no longer maintained or documented.

### Built-in Debugger

Start `node debug script_name.js` to start your script under the builtin
command-line debugger. Your script starts in another Node.js process started with
the `--debug-brk` option, and the initial Node.js process runs the `_debugger.js`
script and connects to your target. See [docs](https://nodejs.org/dist/latest/docs/api/debugger.html) for more information.

### node-inspector

Debug your Node.js app with Chrome DevTools by using an intermediary process
which translates the [Inspector Protocol][] used in Chromium to the V8 Debugger
protocol used in Node.js. See https://github.com/node-inspector/node-inspector for more information.

[Inspector Protocol]: https://chromedevtools.github.io/debugger-protocol-viewer/v8/
[UUID]: https://tools.ietf.org/html/rfc4122
