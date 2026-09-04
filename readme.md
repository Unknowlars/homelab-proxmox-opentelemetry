Configure Proxmox

In the web interface, open **Datacenter → Metric Server → Add →
OpenTelemetry** and use:

| Field | Value |
|---|---|
| Name | `otel-collector` |
| Server | Docker host name or IP |
| Port | `4318` |
| Protocol | `http` |
| Path | `/v1/metrics` |
| Compression | `gzip` |
| Timeout | `5` |
| Verify SSL | off for HTTP |

The equivalent CLI command is:

```bash
pvesh create /cluster/metrics/server/otel-collector \
  --type opentelemetry \
  --server COLLECTOR_HOST \
  --port 4318 \
  --otel-protocol http \
  --otel-path /v1/metrics \
  --otel-compression gzip \
  --otel-timeout 5 \
  --otel-verify-ssl 0
```

The setting is cluster-wide. Configure it once per Proxmox cluster.

For HTTPS, use a trusted reverse proxy or TLS endpoint in front of the
Collector, set `--otel-protocol https`, and keep certificate verification on.

## 5. Verify data flow

On Proxmox:

```bash
cat /etc/pve/status.cfg
journalctl -u pvestatd -n 50
```

On the Docker host:

```bash
curl -fsS http://127.0.0.1:13133
curl -fsS http://127.0.0.1:8888/metrics \
  | grep otelcol_receiver_accepted_metric_points

make discover VM_URL=http://127.0.0.1:8428
make verify VM_URL=http://127.0.0.1:8428
```

Raw discovery output goes to ignored `discovery/local/`. It contains real
infrastructure identifiers and must not be committed.