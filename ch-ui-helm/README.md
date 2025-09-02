# CH-UI Helm Chart

This Helm chart deploys [CH-UI](https://github.com/caioricciuti/ch-ui), a web interface for ClickHouse database management.

## Installation

```bash
helm install ch-ui ./ch-ui-helm
```

## Configuration Options

The chart supports two methods for configuring ClickHouse connection:

### Method 1: Direct Configuration (Default)

Provide ClickHouse connection details directly in values:

```yaml
config:
  clickhouse:
    url: "http://clickhouse:8123"
    user: "default"
    password: "mypassword"
```

Or via command line:

```bash
helm install ch-ui ./ch-ui-helm \
  --set config.clickhouse.url=http://my-clickhouse:8123 \
  --set config.clickhouse.user=myuser \
  --set config.clickhouse.password=mypassword
```

### Method 2: Using Existing Secret (Recommended for Production)

Reference an existing Kubernetes secret containing ClickHouse credentials:

1. First, create a secret with your ClickHouse credentials:

```bash
kubectl create secret generic clickhouse-secret \
  --from-literal=VITE_CLICKHOUSE_URL=http://clickhouse:8123 \
  --from-literal=VITE_CLICKHOUSE_USER=myuser \
  --from-literal=VITE_CLICKHOUSE_PASS=mypassword
```

2. Then reference it in your values:

```yaml
config:
  clickhouse:
    existingSecret: "clickhouse-secret"
    existingSecretKeys:
      url: "VITE_CLICKHOUSE_URL"
      user: "VITE_CLICKHOUSE_USER"
      password: "VITE_CLICKHOUSE_PASS"
```

Or via command line:

```bash
helm install ch-ui ./ch-ui-helm \
  --set config.clickhouse.existingSecret=clickhouse-secret
```

If your secret uses different keys, you can customize them:

```yaml
config:
  clickhouse:
    existingSecret: "my-custom-secret"
    existingSecretKeys:
      url: "clickhouse-host"
      user: "clickhouse-username"
      password: "clickhouse-password"
```

## Key Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Image repository | `ghcr.io/caioricciuti/ch-ui` |
| `image.tag` | Image tag | `latest` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `config.clickhouse.url` | ClickHouse URL (when not using existingSecret) | `http://clickhouse:8123` |
| `config.clickhouse.user` | ClickHouse user (when not using existingSecret) | `default` |
| `config.clickhouse.password` | ClickHouse password (when not using existingSecret) | `""` |
| `config.clickhouse.existingSecret` | Name of existing secret with ClickHouse credentials | `""` |
| `config.clickhouse.existingSecretKeys.*` | Keys in the existing secret | See values.yaml |
| `config.app.basePath` | Base path for reverse proxy | `""` |
| `config.app.requestTimeout` | Request timeout in milliseconds | `30000` |
| `config.app.distributed` | Enable distributed mode for clusters | `false` |
| `ingress.enabled` | Enable ingress | `false` |
| `resources.limits` | Resource limits | `{cpu: 500m, memory: 512Mi}` |
| `resources.requests` | Resource requests | `{cpu: 250m, memory: 256Mi}` |

## Examples

### Basic Installation with Inline Credentials

```bash
helm install ch-ui ./ch-ui-helm \
  --set config.clickhouse.url=http://clickhouse:8123 \
  --set config.clickhouse.user=admin \
  --set config.clickhouse.password=secret123
```

### Production Installation with External Secret

```bash
# Create secret
kubectl create secret generic prod-clickhouse \
  --from-literal=VITE_CLICKHOUSE_URL=https://clickhouse.prod:8443 \
  --from-literal=VITE_CLICKHOUSE_USER=prod-user \
  --from-literal=VITE_CLICKHOUSE_PASS=prod-password

# Install chart
helm install ch-ui ./ch-ui-helm \
  --set config.clickhouse.existingSecret=prod-clickhouse \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=ch-ui.example.com
```

### Installation with Custom Resources and Replicas

```bash
helm install ch-ui ./ch-ui-helm \
  --set replicaCount=3 \
  --set resources.limits.cpu=1000m \
  --set resources.limits.memory=1Gi \
  --set config.clickhouse.existingSecret=clickhouse-creds
```

## Uninstallation

```bash
helm uninstall ch-ui
```