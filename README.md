# OpenSearch Kubernetes Deployment

This repository contains Kubernetes manifests for deploying OpenSearch.

## Components

1. **secret.yaml**: Contains the admin password and credentials
2. **storage.yaml**: Defines persistent storage for OpenSearch data
3. **opensearch.yaml**: Contains the StatefulSet and Service definitions

## Deployment Instructions

1. Create the namespace (optional):
   ```bash
   kubectl create namespace opensearch
   ```

2. Apply the manifests:
   ```bash
   kubectl apply -f opensearch-secret.yaml
   kubectl apply -f opensearch-storage.yaml
   kubectl apply -f opensearch-statefullset.yaml
   ```

3. Check the deployment status:
   ```bash
   kubectl get pods -l app=opensearch
   kubectl get services -l app=opensearch
   ```

4. Access OpenSearch:
   OpenSearch is exposed directly on your host machine via NodePort:
   
   - **OpenSearch API**: http://localhost:30920
   - **Health Check**: http://localhost:30920/_cluster/health
   - **Cluster Info**: http://localhost:30920/_cluster/stats
   
   You can also verify the service is running:
   ```bash
   kubectl get svc opensearch-external
   curl http://localhost:30920/_cluster/health
   ```

5. Login with default credentials:
   - Username: admin
   - Password: admin123 (from secret.yaml)

## Quick Start

After deployment, test your OpenSearch instance:

```bash
# Check cluster health
curl http://localhost:30920/_cluster/health?pretty

# Get cluster info
curl http://localhost:30920/

# Create a test index
curl -X PUT http://localhost:30920/test-index

# Add a document
curl -X POST http://localhost:30920/test-index/_doc \
  -H 'Content-Type: application/json' \
  -d '{"message": "Hello OpenSearch!", "timestamp": "2024-01-01T12:00:00"}'

# Search documents
curl http://localhost:30920/test-index/_search?pretty
```

## Networking Architecture

The deployment creates two services:

1. **opensearch** (ClusterIP): Internal cluster communication on port 9200/9300
2. **opensearch-external** (NodePort): External access via host port 30920
   - Maps host port 30920 to container port 9200
   - No port forwarding required
   - Direct access from your local machine

**Note**: The security plugin is disabled (`DISABLE_SECURITY_PLUGIN: true`) for simplified development access. All API endpoints use HTTP instead of HTTPS.

## Security Considerations

- Change the default admin password in `secret.yaml` before deployment
- Consider enabling the security plugin for production environments
- Implement network policies to restrict access
- Use TLS/HTTPS for production deployments

## Scaling

To scale OpenSearch cluster, increase the replica count in the StatefulSet:
```bash
kubectl scale statefulset opensearch --replicas=3
```

## Troubleshooting

If pods fail to start, check the logs:
```bash
kubectl logs -f opensearch-0
```

Common issues:
- Memory limits too low: Increase resources in opensearch.yaml
- Insufficient storage: Check PVC/PV status
- Network connectivity: Verify service endpoints

