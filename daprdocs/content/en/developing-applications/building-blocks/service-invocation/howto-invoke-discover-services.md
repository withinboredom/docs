---
type: docs
title: "How-To: Invoke and discover services"
linkTitle: "How-To: Invoke services"
description: "How-to guide on how to use Dapr service invocation in a distributed application"
weight: 2000
---

This article describe how to deploy services each with an unique application ID, so that other services can discover and call endpoints on them using service invocation API.

## Prerequisites
Start with the [Dapr getting started guide]({{< ref getting-started >}}) to get install the Dapr CLI and init the Dapr runtime.

## Step 1: Setup a simple service

The following is a Python example of a cart app. It can be written in any programming language. Note that any program or container will work, as long as it exposes an HTTP endpoint.

Save the following as `app.py`:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/add', methods=['POST'])
def add():
    return "Added!"

if __name__ == '__main__':
    app.run()
```

This Python app exposes an `add()` method via the `/add` endpoint.

## Step 2: Choose an ID for your service

Dapr allows you to assign a global, unique ID for your app. This ID encapsulates the state for your application, regardless of the number of instances it may have.

The above example application uses Flask, which defaults to serving HTTP traffic on port 5000. For other applications, make sure to set the `app-port` to the correct value.

{{< tabs "Self-Hosted (CLI)" Kubernetes >}}

{{% codetab %}}
In self hosted mode, set the `--app-id` flag:

```bash
dapr run --app-id cart --app-port 5000 python app.py
```

If your app uses an SSL connection, you can tell Dapr to invoke your app over an insecure SSL connection:

```bash
dapr run --app-id cart --app-port 5000 --app-ssl python app.py
```
{{% /codetab %}}

{{% codetab %}}

### Setup an ID using Kubernetes

In Kubernetes, set the `dapr.io/app-id` annotation on your pod:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
  namespace: default
  labels:
    app: python-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "cart"
        dapr.io/app-port: "5000"
...
```
*If your app uses an SSL connection, you can tell Dapr to invoke your app over an insecure SSL connection with the `app-ssl: "true"` annotation (full list [here]({{< ref kubernetes-annotations.md >}}))*

{{% /codetab %}}

{{< /tabs >}}

## Step 3: Invoke the service

Dapr uses a sidecar, decentralized architecture. To invoke an application using Dapr, you can use the `invoke` API on any Dapr instance.

The sidecar programming model encourages each applications to talk to its own instance of Dapr. The Dapr instances discover and communicate with one another.

{{< tabs curl CLI "VS Code Extension" >}}

{{% codetab %}}
Dapr defaults to using port 3500 for its HTTP API. If you override this value when launching your Dapr app make sure to update it here.

From a command prompt or terminal run:
```bash
curl http://localhost:3500/v1.0/invoke/cart/method/add -X POST
```

Since the add endpoint is a 'POST' method, we used `-X POST` in the curl command.

Dapr puts any payload returned by the called service in the HTTP response's body.
{{% /codetab %}}

{{% codetab %}}
From a command prompt or terminal run:
```bash
dapr invoke --app-id cart --method add
```
{{% /codetab %}}

{{% codetab %}}
The [Visual Studio Code Dapr extension]({{< ref "vscode.md#extension" >}}) supports click-to-invoke for running Dapr applications.
1. Select the Dapr extension in the left nav-bar of VSCode
1. Right click your running application (`cart` in the example app)
1. Select `Invoke (POST) Application Method`
  <br /><img src="/images/service-invocation-vscode-invoke.png" alt="Screenshot of the Dapr VSCode extension invoke option" width="300">
1. For the application method enter `add` and for the payload leave it blank
  <br /><img src="/images/service-invocation-vscode-invoke-method.png" alt="Screenshot of the Dapr VSCode extension invoke method" width="700">
  <br />
  <br /><img src="/images/service-invocation-vscode-invoke-payload.png" alt="Screenshot of the Dapr VSCode extension invoke payload" width="700">
{{% /codetab %}}

{{< /tabs >}}

{{% alert title="Namspaces" color="primary" %}}
When running on [namespace supported platforms]({{< ref "service_invocation_api.md#cross-namespace-invocation" >}}), you include the namespace of the target app in the app ID: `app-id.namespace`

For example, invoking the example python service with a namespace would be:

```bash
curl http://localhost:3500/v1.0/invoke/cart.production/method/add -X POST
```

See the [Cross namespace API spec]({{< ref "service_invocation_api.md#cross-namespace-invocation" >}}) for more information on namespaces.
{{% /alert %}}

## Next steps

### Try out publish & subscribe

Visit the [publish/subscribe docs]({{< ref pubsub >}}) to learn about and get started with Dapr publish & subscribe capabilities.

### View traces and logs

The example above showed you how to directly invoke a different service running locally or in Kubernetes. Dapr outputs metrics, tracing and logging information allowing you to visualize a call graph between services, log errors and optionally log the payload body.

For more information on tracing and logs see the [observability]({{< ref observability-concept.md >}}) article.

 ## Related links
 
- [Service invocation overview]({{< ref service-invocation-overview.md >}})
- [Service invocation API specification]({{< ref service_invocation_api.md >}})
