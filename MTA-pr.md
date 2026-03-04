# Testing MTA Resource Pull Request

This guide explains how to test the MTA (Multi-Target Application) resource feature from PR #5.

## Overview

PR #5 adds support for MTA resources to the Crossplane Provider for Cloud Foundry. You can test this feature before it's merged by using the automatically built container image.

## Pull Request Details

- **GitHub PR**: https://github.com/SAP/crossplane-provider-cloudfoundry/pull/5
- **Branch**: `feat/mta-resource`
- **Related Repository**: https://github.tools.sap/cloud-orchestration/crossplane-provider-mta

## Prerequisites

- A Kubernetes cluster with Crossplane installed (v1.x)
- `kubectl` configured to access your cluster
- Appropriate Cloud Foundry credentials

## Installation

### Step 1: Install the Provider

Create a file named `provider-mta-test.yaml` with the following content:

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-cf-mta-test
spec:
  package: ghcr.io/sap/crossplane-provider-cloudfoundry/crossplane/provider-cloudfoundry:v0.0.0-ac31ff7334784f8bbb3fafa68c0c995240dcaa9c
  packagePullPolicy: IfNotPresent
```

Apply it to your cluster:

```bash
kubectl apply -f provider-mta-test.yaml
```

### Step 2: Verify Installation

Check that the provider is installed and healthy:

```bash
# Check provider status
kubectl get providers

# Check provider pods
kubectl get pods -n crossplane-system

# View provider logs (if needed)
kubectl logs -n crossplane-system -l pkg.crossplane.io/provider=provider-cf-mta-test
```

### Step 3: Configure Provider Credentials (optional if not done before)

Create a secret with your Cloud Foundry credentials:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cf-credentials
  namespace: crossplane-system
type: Opaque
stringData:
  credentials: |
    {
      "api_url": "https://api.cf.eu12.hana.ondemand.com",
      "username": "your-username",
      "password": "your-password"
    }
```

Create a ProviderConfig:

```yaml
apiVersion: cloudfoundry.crossplane.io/v1alpha1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: cf-credentials
      key: credentials
```

Apply both:

```bash
kubectl apply -f cf-credentials-secret.yaml
kubectl apply -f provider-config.yaml
```

## Testing MTA Resources

Once the provider is installed and configured, you can create MTA resources. Check the PR description and examples in the `feat/mta-resource` branch for available resource specifications.

### Example Usage

```yaml
apiVersion: cloudfoundry.crossplane.io/v1alpha1
kind: Mta
metadata:
  name: test
  namespace: default
spec:
  forProvider:
    namespace: default
    spaceRef: 
      name: my-space
    blueGreenDeploy: true
    file:
      url: <url>
      credentialsSecretRef:
        name: secret
        namespace: default
    extension: |
      _schema-version: 3.3.0
      extends: my-app
      ID: my-app.extension
      modules:
        - name: my-app-srv
          parameters:
            instances: 2
```

## Troubleshooting

### Resource Issues

```bash
# Check MTA resource status
kubectl describe mta <resource-name>

# View provider logs
kubectl logs -n crossplane-system -l pkg.crossplane.io/provider=provider-cf-mta-test -f
```

## Providing Feedback

After testing the MTA resource feature, please provide feedback on PR #5:

1. Comment on https://github.com/SAP/crossplane-provider-cloudfoundry/pull/5
2. Include:
   - What MTA scenarios you tested
   - Environment details (Crossplane version, CF environment)
   - Any issues or errors encountered
   - Successful use cases

## Cleanup

When finished testing:

```bash
# Delete any MTA resources
kubectl delete mta --all

# Delete the provider
kubectl delete provider provider-cf-mta-test

