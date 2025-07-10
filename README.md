# Buffer Over-provisioning for Karpenter

A simple Helm chart to maintain "warm" nodes in your Kubernetes cluster with Karpenter, eliminating deployment delays caused by node provisioning.

## 🎯 Problem Solved

With Karpenter, new deployments can be delayed because Karpenter needs to:
1. Detect capacity needs
2. Provision new EC2 nodes
3. Join them to the cluster
4. Schedule pods on them

**Result:** Messages like `0/X nodes available` and 2-3 minute delays.

## 💡 Solution

This chart deploys low-priority "pause" pods that:
- Reserve capacity on existing nodes
- Get automatically evicted when real workloads arrive
- Keep nodes "warm" and ready to use

## 📦 Installation

```bash
# Basic installation
helm install buffer ./buffer-overprovision-chart

# Custom installation
helm install buffer ./buffer-overprovision-chart \
  --set replicaCount=5 \
  --set resources.requests.cpu=1000m \
  --set resources.requests.memory=1Gi
```

## ⚙️ Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of buffer pods | `5` |
| `resources.requests.cpu` | CPU reserved per pod | `500m` |
| `resources.requests.memory` | Memory reserved per pod | `512Mi` |
| `priorityClassName` | PriorityClass name | `overprovision-low` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `extraAffinity` | Additional affinity rules | `{}` |

## 🏗️ Architecture

The chart deploys:
1. **PriorityClass**: Priority -1 (lowest possible)
2. **Deployment**: Pods with anti-affinity to maximize distribution
3. **Pause pods**: Minimal containers that do nothing

## 📊 Usage Examples

### Small cluster (dev/staging)
```bash
helm install buffer ./buffer-overprovision-chart \
  --set replicaCount=2 \
  --set resources.requests.cpu=500m
```

### Production cluster
```bash
helm install buffer ./buffer-overprovision-chart \
  --set replicaCount=10 \
  --set resources.requests.cpu=2000m \
  --set resources.requests.memory=4Gi
```

## 🔍 Verification

```bash
# View buffer pods
kubectl get pods -l app.kubernetes.io/name=buffer-overprovision

# Check distribution across nodes
kubectl get pods -l app.kubernetes.io/name=buffer-overprovision -o wide

# Watch eviction during deployment
kubectl logs -n karpenter -l app.kubernetes.io/name=karpenter
```

## 💰 Cost Considerations

- **Additional cost**: You pay for reserved capacity
- **Savings**: Reduced deployment time and better developer experience
- **Optimization**: Adjust `replicaCount` based on your deployment patterns

## 🎯 Best Practices

1. **Start small**: 2-3 buffer pods and increase as needed
2. **Monitor**: Track how often your buffer pods get evicted
3. **Adjust**: If frequent eviction → increase buffer count
4. **Cost vs Performance**: Find your optimal balance

## 🚀 Updating

```bash
# Increase buffer capacity
helm upgrade buffer ./buffer-overprovision-chart \
  --set replicaCount=10

# Reduce during off-hours
helm upgrade buffer ./buffer-overprovision-chart \
  --set replicaCount=2
```

## 🗑️ Uninstallation

```bash
helm uninstall buffer
```