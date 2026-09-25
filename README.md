# bw5_greeting

A small **TIBCO BusinessWorks 5 (BW5)** project that exposes an HTTP endpoint. When
called, it returns a styled HTML "status card" that greets the caller by name and
reports details about the BusinessWorks engine it is running on — the host name, the
BW engine version, and the application version.

It's handy as a quick smoke-test / health-check service to confirm that a BW5 engine
is up and to see at a glance which version and host is serving a request.

## What it does

The single process, `Processes/Greeting.process`, runs this flow:

```
HTTP Receiver ──▶ Log ──▶ Get BW Environment ──▶ Send HTTP Response ──▶ End
```

1. **HTTP Receiver** — an HTTP event source that listens on the shared HTTP
   connection and accepts incoming requests.
2. **Log** — writes `Received Message: <query string>` to the BW log.
3. **Get BW Environment** — a Java activity that resolves:
   - `hostName` — the local machine's host name.
   - `bwVersion` — the BW engine version, discovered from (in order) the
     `BW_VERSION` property/env var, the `TIBCO_INTERNAL_BW5_BUILDTYPE_TAG` image
     tag, the `engine.jar` path combined with `TIBCO_HOME/_installInfo`, and
     finally the engine package metadata.
   - `appVersion` — from `APP_VERSION` / `BW_PROJECT_VERSION`, defaulting to
     `Development / Local`.
4. **Send HTTP Response** — returns an HTML page (`text/html; charset=UTF-8`) with a
   card showing `Hello <name>` and a table of Host / BW Version / App Version.

## Usage

The name shown in the greeting comes from the request **query string**. For example,
if the service is running on the default port:

```
http://localhost:8282/?World
```

produces a page greeting **"Hello World"** along with the engine details.

## Configuration

Global variables live in `defaultVars/defaultVars.substvar`. The most relevant one:

| Variable   | Default | Description                          |
| ---------- | ------- | ------------------------------------ |
| `httpPort` | `8282`  | Port the HTTP connection listens on. |

The HTTP listener is defined in `Shared Connections/HTTP Connection.sharedhttp`
(host `localhost`, port `%%httpPort%%`, SSL disabled).

The BW version reporting can be influenced at runtime via environment variables /
system properties such as `BW_VERSION`, `TIBCO_HOME`, `APP_VERSION`, and
`BW_PROJECT_VERSION` (see the Java activity in `Processes/Greeting.process`).

## Project layout

| Path                                       | Description                                        |
| ------------------------------------------ | -------------------------------------------------- |
| `Processes/Greeting.process`               | The main HTTP service process.                     |
| `Shared Connections/HTTP Connection.sharedhttp` | Shared HTTP listener configuration.           |
| `defaultVars/defaultVars.substvar`         | Global (deployment-settable) variables.            |
| `AESchemas/`                               | Referenced AE / XSD schema definitions.            |
| `EAR/bw5_greeting.ear`                     | Pre-built enterprise archive for deployment.       |
| `bw5_greeting.archive`                     | EAR/archive build descriptor.                      |
| `vcrepo.dat`                               | BW project metadata (design-time version 5.12.3).  |

## Building & running

This is a TIBCO BusinessWorks 5 project, so it is developed and run with the TIBCO
toolset rather than a generic build tool:

1. Open the project folder in **TIBCO Designer**.
2. Run the `Greeting` process in the **Designer Tester**, or deploy the prebuilt
   `EAR/bw5_greeting.ear` to a BW engine via TIBCO Administrator / `AppManage`.
3. Set `httpPort` at deployment time if the default `8282` is not suitable.
4. Browse to `http://<host>:<httpPort>/?<name>` to see the greeting.

> Requires a TIBCO BusinessWorks 5.x runtime (project authored with 5.12.3).
