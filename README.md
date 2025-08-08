# Bookinfo Ratings Service - Kubernetes & Helm

This repository contains the necessary Kubernetes manifests and Helm charts to deploy the **Bookinfo Ratings service**. It includes scripts for database initialization and environment-specific configurations for `dev`, `uat`, and `prd`.

## Directory Structure

```
.
├── databases/
│   ├── ratings_data.json  # Sample ratings data for MongoDB.
│   └── script.sh          # Script to import data into MongoDB.
├── k8s/
│   ├── helm/              # Helm chart for the Ratings service.
│   │   ├── Chart.yaml
│   │   └── templates/
│   │       ├── ratings-deployment.yaml
│   │       ├── ratings-ingress.yaml
│   │       └── ratings-service.yaml
│   └── helm-values/       # Environment-specific Helm values.
│       ├── values-bookinfo-dev-ratings.yaml
│       ├── values-bookinfo-prd-ratings.yaml
│       └── values-bookinfo-uat-ratings.yaml
└── README.md              # This documentation.
```

## Prerequisites

Before you begin, ensure you have the following:

- A running Kubernetes cluster (v1.19+ recommended).
- `kubectl` configured to connect to your cluster.
- Helm v3.x installed.
- Access to a container registry where the service image is stored.
- MongoDB secrets created in the target namespace.

## Getting Started

Follow these steps to deploy the Ratings service and its MongoDB backend.

### Step 1: Prepare Secrets

The application requires MongoDB credentials to be stored in a Kubernetes Secret. If you haven't created them, apply the secret manifest to your target namespace.

*Example command:*
```sh
# Replace with the actual path to your secret definition file.
kubectl apply -f /path/to/your/bookinfo-secret.yaml -n <namespace>
```

### Step 2: Deploy MongoDB

This step deploys a MongoDB instance using Helm.

#### 2a. (Optional) Create ConfigMap for Data Initialization

To initialize MongoDB with sample data, create a `ConfigMap` from the files in the `databases/` directory. This `ConfigMap` will be mounted into the MongoDB pod to run the initialization script.

*For the `dev` environment:*
```sh
kubectl create configmap bookinfo-dev-ratings-mongodb-initdb \
  --from-file=databases/ratings_data.json \
  --from-file=databases/script.sh \
  -n bookinfo-dev
```
*Note: Adjust the namespace (`-n`) and `ConfigMap` name for other environments (`uat`, `prd`).*

#### 2b. Install the MongoDB Helm Chart

Deploy MongoDB using the appropriate values file for your environment.

*For the `dev` environment:*
```sh
helm install bookinfo-dev-ratings-mongodb bitnami-pre-2022/mongodb \
  --version 10.0.1 \
  -f k8s/helm-values/values-bookinfo-dev-ratings-mongodb.yaml \
  -n bookinfo-dev
```
*Note: Ensure you have the `bitnami-pre-2022` Helm repository added.*

### Step 3: Deploy the Ratings Service

Once MongoDB is running, deploy the Ratings service using its Helm chart and the corresponding environment values.

*For the `dev` environment:*
```sh
helm install bookinfo-dev-ratings k8s/helm \
  -f k8s/helm-values/values-bookinfo-dev-ratings.yaml \
  -n bookinfo-dev
```
*Replace the values file and namespace for `uat` or `prd` deployments.*

## Accessing the Service

The service is exposed via a Kubernetes Ingress. To access it, you may need to update your DNS records or add an entry to your local `/etc/hosts` file to point the hostname to your Ingress controller's IP address.

- **Example URL:** `http://<your-host>/ratings/`

Check the Ingress resource for the configured hostname:
```sh
kubectl get ingress -n <namespace>
```

## Configuration

To customize the deployment for different environments, modify the following files:

- **Application & Resource Settings:** Edit the `values-bookinfo-*-ratings.yaml` files in `k8s/helm-values/`.
- **MongoDB Settings:** Edit the `values-bookinfo-*-ratings-mongodb.yaml` files in `k8s/helm-values/`.
- **Initial Data:** Modify `databases/ratings_data.json` to change the sample data loaded into MongoDB.

## Troubleshooting

If you encounter issues, here are some initial troubleshooting steps:

- **Check Pod Status:**
  ```sh
  kubectl get pods -n <namespace>
  ```
- **View Pod Logs:**
  ```sh
  kubectl logs -n <namespace> deployment/bookinfo-ratings
  ```
- **Describe Pods for Events:**
  ```sh
  kubectl describe pod <pod-name> -n <namespace>
  ```
- **Verify Secrets:** Ensure that the MongoDB secrets exist in the correct namespace and are correctly mounted as environment variables in the deployment.

## Teardown

To remove the deployed resources, use `helm uninstall`.

- **Uninstall Ratings Service:**
  ```sh
  helm uninstall bookinfo-dev-ratings -n bookinfo-dev
  ```
- **Uninstall MongoDB:**
  ```sh
  helm uninstall bookinfo-dev-ratings-mongodb -n bookinfo-dev
  ```
- **Delete ConfigMap:**
  ```sh
  kubectl delete configmap bookinfo-dev-ratings-mongodb-initdb -n bookinfo-dev
  ```

## References

- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)