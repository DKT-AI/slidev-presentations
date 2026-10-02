# Kubernetes 1.37 queue autoscaling teaching examples

These files are illustrative configuration, not evidence from a running cluster.

- `hpa-queue.yaml`: complete HPA for an existing `default/queue-worker` Deployment. Start that Deployment with at least one replica. The HPA uses `AverageValue: "30"` to teach a queue-items-per-worker target. Unlike the release blog's `Value` example, this yields a baseline desired count of `ceil(total queue items / 30)`, before tolerance, stabilization and scaling limits.
- `adapter-external-rules.yaml`: Prometheus Adapter configuration fragment. Integrate it into your existing adapter configuration; it is not a resource that can be applied directly with kubectl.

Prerequisites: Prometheus already scrapes `queue_consumer_lag{namespace="default",name="worker_tasks"}` from a source that survives zero worker replicas. An empty queue must produce a numeric zero, not a missing time series. A configured adapter and External Metrics API registration must make the selected metric available. Kubernetes API server and controller manager must both support and enable `HPAScaleToZero`.

Do not run two independent autoscalers against the same Deployment. Validate read-only metric queries before introducing the HPA. Before disabling the feature or rolling back, set affected HPAs to `minReplicas >= 1` and wake targets currently at zero. Review workload shutdown and queue acknowledgement semantics separately.

## Official provenance

- https://kubernetes.io/blog/2026/09/02/kubernetes-v1-37-hpa-scale-to-zero-beta/
- https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/
- https://www.kubernetes.dev/resources/keps/2021/
- https://github.com/kubernetes-sigs/prometheus-adapter/blob/master/docs/externalmetrics.md
- https://github.com/kubernetes/kubernetes/blob/v1.37.0/pkg/controller/podautoscaler/replica_calculator.go

Prepared 2026-10-02. No cluster changes or dependency installations are included.
