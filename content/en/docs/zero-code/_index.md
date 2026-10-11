---
title: Zero-code Instrumentation
redirects: [{ from: 'net/*', to: 'dotnet/:splat' }]
weight: 265
---

OpenTelemetry [zero-code instrumentation][] adds telemetry to your application
without changes to your code. Choose the option that matches where your
application runs:

- **On a host, a virtual machine, or in a container that you start yourself:**
  Follow the guide for your language in the section index below.
- **In Kubernetes:** Use the [OpenTelemetry Operator for Kubernetes][otel-op] to
  [inject zero-code instrumentation][] for .NET, Java, Node.js, Python, or Go.
  The Operator injects and configures the instrumentation for you.

Whichever option you choose, check that your application sends telemetry before
you continue. For an example, see [Verify that it works][verify] in the
JavaScript guide.

[inject zero-code instrumentation]:
  /docs/platforms/kubernetes/operator/automatic/
[verify]: js/#verify-that-it-works
[zero-code instrumentation]: /docs/concepts/instrumentation/zero-code/
[otel-op]: /docs/platforms/kubernetes/operator/
