# OpenShift Compatible

This example shows the modifications from the [default](../default-values) to deploy Kubernetes Monitoring on an OpenShift cluster.

These modifications prevent deploying Kube State Metrics and Node Exporter, since they will already be present on the
cluster, and adjust the configuration to the Grafana Agent to find those existing components.
It also modifies the Grafana Agent to lock down security and permissions, and assigns a high-number port. 

The `platform: openshift` switch also creates a SecurityContextConstraints object that modifies the permissions for the
Grafana Agent for logs. This is required because of its use of hostPath volume mounts to detect and capture pod logs.

Note that these agent pods cannot enable `readOnlyRootFilesystem` because they require being able to write to their
storage path, which defaults to `/tmp/agent`.

```yaml
cluster:
  name: openshift-compatible-test
  platform: openshift

externalServices:
  prometheus:
    host: https://prometheus.example.com
    proxyURL: http://192.168.1.100:8080
    basicAuth:
      username: "12345"
      password: "It's a secret to everyone"

  loki:
    host: https://prometheus.example.com
    proxyURL: http://192.168.1.100:8080
    basicAuth:
      username: "12345"
      password: "It's a secret to everyone"

metrics:
  kube-state-metrics:
    service:
      port: https-main
      isTLS: true

  node-exporter:
    labelMatchers:
      app.kubernetes.io/name: node-exporter
    service:
      isTLS: true

kube-state-metrics:
  enabled: false

prometheus-node-exporter:
  enabled: false

grafana-agent:
  agent:
    listenPort: 8080
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: [ "ALL" ]
      runAsNonRoot: true
      seccompProfile:
        type: "RuntimeDefault"

grafana-agent-logs:
  agent:
    listenPort: 8080
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: [ "ALL" ]
      privileged: false
      runAsUser: 0
  global:
    podSecurityContext:
      seLinuxOptions:
        type: spc_t
```
