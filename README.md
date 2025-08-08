# Bookinfo Ratings Service - Kubernetes & Helm

This directory contains Kubernetes manifests and Helm charts for deploying the Bookinfo Ratings service, along with MongoDB initialization scripts and environment-specific configuration.

## Structure

- `databases/`
  - `ratings_data.json`: Sample ratings data for MongoDB initialization.
  - `script.sh`: Script to import ratings data into MongoDB.
- `k8s/helm/`: Helm chart for the Ratings service.
  - `Chart.yaml`: Helm chart metadata.
  - `templates/`: Kubernetes resource templates (Deployment, Service, Ingress).
- `k8s/helm-values/`: Environment-specific Helm values files (dev, uat, prd).
- `README.md`: This documentation.

## Prerequisites

- Kubernetes cluster (v1.19+ recommended)
- Helm 3.x
- Access to a container registry (for pulling images)
- MongoDB secrets created in the target namespace

## Usage

### 1. Prepare Secrets

Create the required MongoDB secrets in each namespace (see examples in `/Users/prakarnj/web_dev/k8s/bookinfo-secret/`).

```sh
kubectl apply -f /path/to/bookinfo-secret/
```

### 1a. (Optional) Create ConfigMap for MongoDB Initialization

If you want to initialize MongoDB with sample data and a script, create a ConfigMap with the following command:

```sh
kubectl create configmap bookinfo-dev-ratings-mongodb-initdb \
  --from-file=databases/ratings_data.json \
  --from-file=databases/script.sh
```

Make sure to update the namespace or ConfigMap name as needed for your environment.

### 2. Deploy MongoDB (if needed)

Use your preferred MongoDB Helm chart, passing the appropriate values file, e.g.:

```sh
helm install bookinfo-dev-ratings-mongodb bitnami-pre-2022/mongodb --version 10.0.1 -f k8s/helm-values/values-bookinfo-dev-ratings-mongodb.yaml
```

### 3. Deploy Ratings Service

```sh
helm install bookinfo-dev-ratings k8s/helm -f k8s/helm-values/values-bookinfo-dev-ratings.yaml
```

Replace `bookinfo-dev` and values file as needed for other environments.

### 4. Access the Service

- The service is exposed via Ingress. Update your DNS or `/etc/hosts` if needed.
- Example URL: `http://<host>/ratings/`

## Customization

- Modify values in `k8s/helm-values/` for different environments.
- Update `databases/ratings_data.json` for initial MongoDB data.

## Troubleshooting

- Check pod logs:  
  `kubectl logs -n <namespace> deployment/bookinfo-ratings`
- Verify secrets and environment variables are set correctly.

## References

- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
