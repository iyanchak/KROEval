# KRO Evaluation & Implementation Guide

This project demonstrates the implementation of a custom internal developer platform (IDP) using **KRO**, abstracting complex Kubernetes resources into simple, high-level Custom Resources.

## 🚀 Deployment Summary

The implementation was transitioned to the official KRO OCI registry and updated to handle standard Kubernetes API requirements.

### Key Changes Implemented
1.  **Official Chart Integration**: Migrated from local operator files to the official KRO chart using `oci://registry.k8s.io/kro/charts/kro`.
2.  **Image Updates**:
    * **KRO Controller**: Updated to `v0.9.1`.
    * **Sample App**: Configured to use a local registry image: `localhost:32000/counter-server:latest`.
3.  **Value Alignment**: Adjusted `values.yaml` to include mandatory keys for `metrics`, `serviceAccount`, `metadata`, `debug`, and `config` to ensure compatibility with the official chart templates.
4.  **Schema Evolution**: Implemented version bumping (`v1alpha1` → `v1alpha4`) to handle breaking changes in the ResourceGraphDefinition (RGD) without requiring manual CRD deletions.

---

## 📦 Packaged Artifacts
The following Helm charts were linted and packaged:
* `kro-crds-1.0.0.tgz` — The base Custom Resource Definitions.
* `kro-v0.9.1.tgz` — Unused and ignore
* `rgd-definition-1.0.0.tgz` — The `ResourceGraphDefinition` (Deployment + Service template).
* `app-instance-1.0.0.tgz` — The final user-facing application instance.

---

## 🛠 Installation Sequence (MicroK8s)

To ensure the dependency chain is respected and avoid "AlreadyExists" errors, follow this specific order.

### 1. Engine & Foundation
```bash
# Install CRDs
helm install kro-crds ./kro-crds-1.0.0.tgz

# Install KRO Controller (Engine) using Upgrade-Install
helm upgrade --install kro oci://registry.k8s.io/kro/charts/kro \\
  --version 0.8.5 \\
  --set image.tag=v0.8.5
2. Definition & Application
Bash
# Install the RGD (Recipe)
helm install rgd-definition ./rgd-definition-1.0.0.tgz

# Install the App Instance (Actual deployment)
helm install app-instance ./app-instance-1.0.0.tgz
🔍 Verification & Testing
Confirming the Resource Graph
Verify that KRO has established the new TestApp API:

Bash
kubectl get crd testapps.demo.kro.run
Testing the Application
The sample application (counter-server) was successfully verified using curl against the assigned NodePort:

Bash
# Get the NodePort
kubectl get svc my-app-instance-svc

# Check hits (Example NodePort <NODE_PORT>)
curl -v [http://<NODE_IP>:<NODE_PORT>/hits](http://<NODE_IP>:<NODE_PORT>/hits)

# Increment hits
curl -X POST [http://<NODE_IP>:<NODE_PORT>/hits](http://<NODE_IP>:<NODE_PORT>/hits)