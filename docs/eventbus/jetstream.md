## Jetstream

[Jetstream](https://docs.nats.io/nats-concepts/jetstream) is the latest streaming server implemented by the NATS community, with improvements from the original NATS Streaming (which will eventually be deprecated).

A simplest Jetstream EventBus example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: EventBus
metadata:
  name: default
spec:
  jetstream:
    version:
      latest # Do NOT use "latest" but a specific version in your real deployment
      # See: https://argoproj.github.io/argo-events/eventbus/jetstream/#version
```

The example above brings up a Jetstream
[StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
with 3 replicas in the namespace.

## Properties

Check
[here](../APIs.md#argoproj.io/v1alpha1.JetStreamBus)
for the full spec of `jetstream`.

### version

The version number specified in the example above is the release number for the NATS server. We will support some subset of these as we've tried them out and only plan to upgrade them as needed. The list of available versions is managed by the controller manager ConfigMap, which can be updated to support new versions.

```
kubectl get configmap argo-events-controller-config -o yaml
```

Check [here](https://docs.nats.io/nats-concepts/jetstream/streams#configuration) for a list of configurable features per version.

### A more involved example

Another example with more configuration:

```
apiVersion: argoproj.io/v1alpha1
kind: EventBus
metadata:
  name: default
spec:
  jetstream:
    version: latest # Do NOT use "latest" but a specific version in your real deployment
    replicas: 5
    persistence: # optional
        storageClassName: standard
        accessMode: ReadWriteOnce
        volumeSize: 10Gi
    streamConfig: |             # see default values in argo-events-controller-config
      maxAge: 24h
    settings: |
      max_file_store: 1GB       # see default values in argo-events-controller-config
    startArgs:
      - "-D"                    # debug-level logs
```

## Security

For Jetstream, TLS is turned on for all client-server communication as well as between Jetstream nodes. In addition, for client-server communication we by default use password authentication (and because TLS is turned on, the password is encrypted).

## How it works under the hood

Jetstream has the concept of a Stream, and Subjects (i.e. topics) which are used on a Stream. From the documentation: “Each Stream defines how messages are stored and what the limits (duration, size, interest) of the retention are.” For Argo Events, we have one Stream called "default" with a single set of settings, but we have multiple subjects, each of which is named `default.<eventsourcename>.<eventname>`. Sensors subscribe to the subjects they need using durable consumers.

### Exotic

To use an existing JetStream service, follow the example below.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: EventBus
metadata:
  name: default
spec:
  jetstreamExotic:
    url: nats://xxxxx:xxx
    accessSecret:
      name: my-secret-name
      key: secret-key
    streamConfig: ""
```
