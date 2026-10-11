---
title: JavaScript zero-code instrumentation
linkTitle: JavaScript
description:
  Capture telemetry from your application with zero source code modifications
aliases: [/docs/languages/js/automatic]
cSpell:ignore: rolldice
---

Zero-code instrumentation for JavaScript provides a way to instrument any
Node.js application and capture telemetry data from many popular libraries and
frameworks without any code changes.

On this page, you install the instrumentation, run your application with it, and
check that your application sends traces.

## Prerequisites

Before you start, make sure that you have the following:

- A Node.js application that you can run. Use a [supported version of
  Node.js][node-versions]. If you don't have an application, use the [example
  application][] from Getting Started. The examples on this page use it.
- A place to send your telemetry, such as an [OpenTelemetry Collector][] or an
  observability backend. You don't need it for your first check: you can print
  the telemetry to your terminal instead.

## Setup

Run the following commands to install the appropriate packages.

```shell
npm install --save @opentelemetry/api
npm install --save @opentelemetry/auto-instrumentations-node
```

The `@opentelemetry/api` and `@opentelemetry/auto-instrumentations-node`
packages install the API, SDK, and the instrumentation tools.

## Configure and run your application

To start the instrumentation, run your application with the
`--require @opentelemetry/auto-instrumentations-node/register` option. You
configure the instrumentation with environment variables.

The following command sets a service name and sends telemetry to a Collector
that runs on your computer:

```shell
OTEL_SERVICE_NAME="my-service" \
OTEL_EXPORTER_OTLP_ENDPOINT="http://localhost:4318" \
node --require @opentelemetry/auto-instrumentations-node/register app.js
```

Replace `my-service` with the name of your service, and `app.js` with the file
that starts your application.

The command uses the following environment variables:

- `OTEL_SERVICE_NAME`: The name of your service. Your observability backend
  shows your telemetry under this name. If you don't set it, the name is
  `unknown_service:node`.
- `OTEL_EXPORTER_OTLP_ENDPOINT`: The address that receives your telemetry. The
  instrumentation sends the data with OTLP over HTTP. The default value is
  `http://localhost:4318`. The value must start with `http://` or `https://`.

By default, the instrumentation sends traces, metrics, and logs.

> [!NOTE] Signal-specific endpoints
>
> To send only traces to a different address, you can set
> `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT`. This variable uses the URL exactly as
> you write it, so include the full path, for example
> `http://localhost:4318/v1/traces`. With `OTEL_EXPORTER_OTLP_ENDPOINT`, the
> instrumentation adds the path for you.

If you start your application with a script, such as the `start` script in your
`package.json` file, set the `--require` option in `NODE_OPTIONS` instead:

```shell
OTEL_SERVICE_NAME="my-service" \
OTEL_EXPORTER_OTLP_ENDPOINT="http://localhost:4318" \
NODE_OPTIONS="--require @opentelemetry/auto-instrumentations-node/register" \
npm start
```

> [!TIP] Avoid `export` for `NODE_OPTIONS`
>
> If you use `export` to set `NODE_OPTIONS`, the value stays set in your
> terminal. Every `node` command that you run in that terminal then loads the
> instrumentation. In a project without the instrumentation package, this causes
> a `Cannot find module` error.

By default, all SDK [resource detectors](/docs/languages/js/resources/) are
used. You can use the environment variable `OTEL_NODE_RESOURCE_DETECTORS` to
enable only certain detectors, or to completely disable them.

To see the full range of configuration options, see
[Module Configuration](configuration).

## Verify that it works

Your application prints the same messages with or without the instrumentation.
To see if the instrumentation works, check for traces with one of the following
methods.

### Print traces to your terminal

This check doesn't need a Collector.

1. Start your application with the console exporter:

   ```shell
   OTEL_SERVICE_NAME="my-service" \
   OTEL_TRACES_EXPORTER="console" \
   OTEL_METRICS_EXPORTER="none" \
   OTEL_LOGS_EXPORTER="none" \
   node --require @opentelemetry/auto-instrumentations-node/register app.js
   ```

   The values `none` turn off metrics and logs for this check. Nothing receives
   them, and the output stays shorter.

2. Send a request to your application. For example, open one of its pages in
   your browser.

3. Look at the terminal where your application runs.

You know that the instrumentation works when you see a span with these values:

- `name`: The HTTP method and route of your request, for example
  `'GET /rolldice'`.
- `'service.name'`: The service name that you set.

One request can create several spans, for example one from the HTTP server and
one from Express.

> [!NOTE] `MetadataLookupWarning`
>
> You might see the warning `MetadataLookupWarning: received unexpected error`.
> It comes from the resource detector that checks if your application runs on
> Google Cloud. If your application doesn't run on Google Cloud, you can ignore
> this warning. You can also turn off this detector with
> `OTEL_NODE_RESOURCE_DETECTORS`.

<details>
<summary>Example output (shortened)</summary>

```text
{
  resource: {
    attributes: {
      'service.name': 'my-service',
      ...
    }
  },
  instrumentationScope: {
    name: '@opentelemetry/instrumentation-http',
    ...
  },
  traceId: 'ec62d125e9e027121bd74c3c2818b1b4',
  name: 'GET /rolldice',
  kind: 1,
  ...
  attributes: {
    'http.request.method': 'GET',
    'url.path': '/rolldice',
    'http.response.status_code': 200,
    ...
  },
  ...
}
```

</details>

### Send traces to a Collector

1. Start a Collector as described in the [Collector quick start][]. The quick
   start uses Docker. The default configuration prints the telemetry that the
   Collector receives.

2. Start your application with the command from
   [Configure and run your application](#configure-and-run-your-application).

3. Send a request to your application.

4. Wait a few seconds. The instrumentation sends traces in batches.

You know that the instrumentation works when the Collector output shows a span
with these values:

- `Name`: The HTTP method and route of your request, for example
  `GET /rolldice`.
- `service.name`: The service name that you set, under `Resource attributes`.

The Collector also prints its own metrics, so the output gets long quickly. If
you started the Collector as in the quick start, its output is also in the
`collector-output.txt` file. To find your spans, run the following command:

```shell
grep -E 'service.name: Str\(my-service\)|Name +:' collector-output.txt
```

<details>
<summary>Example output (shortened)</summary>

```text
Resource attributes:
     -> service.name: Str(my-service)
     ...
Span #0
    Trace ID       : 85e7447d195113c55ba4598e469fde08
    Parent ID      :
    ID             : c55ae8e57e641dda
    Name           : GET /rolldice
    Kind           : Server
    ...
```

</details>

> [!NOTE] `service.name` in the Collector logs
>
> Each Collector log line contains `"service.name": "otelcol"`. The metrics of
> the Collector also show `service.name: Str(otelcol)` under
> `Resource attributes`. This is the name of the Collector itself, not the name
> of your service.

## Running in Kubernetes

If your application runs in Kubernetes, you can use the [OpenTelemetry Operator
for Kubernetes][operator] to inject zero-code instrumentation. For the steps,
see [Node.js instrumentation with the Operator][operator-nodejs].

> [!NOTE]
>
> The Operator configures the Node.js instrumentation to send data with OTLP
> over gRPC. Use the gRPC port of your Collector, usually `4317`, not `4318`.

## Supported libraries and frameworks

A number of popular Node.js libraries are auto-instrumented. For the full list,
see
[supported instrumentation](https://github.com/open-telemetry/opentelemetry-js-contrib/tree/main/packages/auto-instrumentations-node#supported-instrumentations).

## Troubleshooting

If you don't see any traces, check the following:

- **Does the instrumentation create spans?** First,
  [print traces to your terminal](#print-traces-to-your-terminal). If you see
  spans there, the instrumentation works, and the problem is with sending the
  data.
- **Does the endpoint start with `http://` or `https://`?** If it doesn't, for
  example `localhost:4318`, the instrumentation can't send any data. At the
  default log level, you don't see an error. With `OTEL_LOG_LEVEL` set to
  `debug`, you see `ERR_INVALID_PROTOCOL`.
- **Is the endpoint a URL at all?** If the value of
  `OTEL_EXPORTER_OTLP_ENDPOINT` isn't a URL, such as `my-endpoint`, the
  instrumentation uses the default value `http://localhost:4318` without an
  error.
- **Does a signal-specific endpoint include the path?** If you set
  `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT`, the value must end with `/v1/traces`.
- **Does something receive data at the endpoint?** Set `OTEL_LOG_LEVEL` to
  `debug` and search the output for `ECONNREFUSED`. This error means that
  nothing accepts connections at that address and port.
- **Does your service have the wrong name?** If you see `unknown_service:node`,
  set `OTEL_SERVICE_NAME`.

You can set the log level by setting the `OTEL_LOG_LEVEL` environment variable
to one of the following:

- `none`
- `error`
- `warn`
- `info`
- `debug`
- `verbose`
- `all`

The default level is `info`.

> [!NOTE]
>
> - In a production environment, it is recommended to set `OTEL_LOG_LEVEL` to
>   `info`.
> - Logs are always sent to `console`, no matter the environment or debug level.
> - Debug logs are extremely verbose and can negatively impact the performance
>   of your application. Enable debug logging only when needed.

## Next steps

- To change more settings, see [Module Configuration](configuration).
- To add your own spans and metrics, see
  [code-based instrumentation](/docs/languages/js/instrumentation/).
- To process and route your telemetry, learn more about the [OpenTelemetry
  Collector][].

[Collector quick start]: /docs/collector/quick-start/
[example application]:
  /docs/languages/js/getting-started/nodejs/#example-application
[node-versions]: /docs/languages/js/#version-support
[OpenTelemetry Collector]: /docs/collector/
[operator]: /docs/platforms/kubernetes/operator/
[operator-nodejs]: /docs/platforms/kubernetes/operator/automatic/#nodejs
