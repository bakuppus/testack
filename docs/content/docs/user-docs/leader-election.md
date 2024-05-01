---
title: "Managing multiple instances of ACK with leader election"
description: "Configure leader election for ACK controllers"
draft: false
menu:
  docs:
    parent: "getting-started"
weight: 60
toc: true
---

In a Kubernetes cluster, you may want to run multiple instances of any ACK
controller - configured for different accounts or regions, or for fail state 
rollover. 
However, to avoid conflicts and ensure proper resource management, it's necessary
to designate one instance as the leader, which takes responsibility for executing
certain operations while the other instances remain passive. In the event that
the leader instance fails, [leader election][leader-election] ensures the seamless
transition of leadership to another healthy instance.

By default, leader election is disabled in the ACK Helm charts. However, once
enabled, you gain the flexibility to scale the default deployment of ACK controllers
from a single replica (1) to a higher number.

## Enabling Leader Election for ACK Controllers

To enable leader election when installing an ACK controller, set the
`leaderElection.enabled` to `true` in the helm chart `values.yaml` like:

```yaml
leaderElection:
    enabled: true
```

You also have the flexibility to scale the number of controller replicas. Edit
the `deployment.replicas` configuration in the same `values.yaml` file and
adjust it to your desired count, such as:

```yaml
deployment:
    replicas: 3
```

## Configuring Leader Election `Namespace`

The leader election namespace is a controller configuration setting that
determines the namespace where controllers manage leader election. Under the 
hood it is used for storing `coordination.k8s.io/lease` objects, which help
controllers reach consensus and choose a leader. If not specified, the system
will use the namespace from the service account's configuration by default.

If you would like to set leader election namespace for your ACK controllers,
you need to set `leaderElection.namespace`, like below:

```yaml
leaderElection:
    enabled: true
    namespace: "ack-leader-election-ns"
```

## Verifying Leader Election

To confirm that leader election is active, you can perform the following checks:

- **Log Messages**: Examine the logs of your ACK controller pods for any messages
  indicating the successful execution of leader election.
- **`coordination.k8s.io/lease` Objects**: You can also inspect the
  `coordination.k8s.io/lease` objects within the configured leader election
  namespace. Using `kubectl` you can retrieve information about these objects,
  allowing you to verify leadership status.
- **Kubernetes Events**: Another method is to monitor Kubernetes events related to your
  controllers. Execute `kubectl get events` to view events that might provide insights
  into leader election and controller behavior.

[leader-election]: https://kubernetes.io/docs/concepts/architecture/leases/#leader-election