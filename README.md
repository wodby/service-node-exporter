# Node exporter Kubernetes system service for Wodby

Node Exporter exposes host and node metrics used by the Wodby cluster
monitoring pipeline.

This repository defines the Wodby service manifests and operational
configuration for Node exporter.

- [Wodby Kubernetes platform](https://wodby.com)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby system stacks using this service

- [Monitoring system stack](https://github.com/wodby/stack-monitoring)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `node-exporter` |
| Type | Infrastructure service |
| Workloads | `main` (DaemonSet), primary; fixed replica count |
| Containers | `node-exporter`, `local-path-volume-usage` (sidecar) |
| Helm | chart `prometheus-community/prometheus-node-exporter`; version `4.60.1` |

## Role in Wodby infrastructure

Wodby installs this service through a Kubernetes system stack when it is
required by the cluster provider or selected infrastructure configuration. It
runs as a cluster-owned system app and is not offered as a user-deployable
application service.

## K3S volume usage

On K3S, the built-in `local-path` volumes are folders on the server's disk, and
Kubernetes reports no usage for them. The `local-path-volume-usage` sidecar
measures each folder under `/var/lib/rancher/k3s/storage` every 10 minutes and
node-exporter serves the result through its textfile collector:

| Metric | Labels | Meaning |
| --- | --- | --- |
| `wodby_local_path_volume_used_bytes` | `pvc_namespace`, `persistentvolumeclaim` | Disk space the volume's folder uses |
| `wodby_local_path_volume_scan_timestamp_seconds` | | When the last scan finished |

The claim's namespace is in `pvc_namespace` because the scrape sets `namespace`
to node-exporter's own. The sidecar reads the host through the pod's read-only
root mount and runs as root with only the `DAC_READ_SEARCH` capability, so it
can measure folders owned by other users. On nodes without that folder it
writes nothing.

## Platform maintenance

Changes to this repository can affect cluster provisioning, upgrades,
networking, or observability. Coordinate manifest and Helm changes with every
dependent system stack and preserve service, workload, endpoint, config, and
volume identifiers.

Wodby platform maintainers can validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
