# Crossplane-iac-Nairobi-DevOps-Community

A simple demo on Using crossplane to create infrastructure for the Nairobi DevOps Community

# Prerequisites

- A Kubernetes cluster with at least 2 GB of RAM (Consider Minikube/Kind)
- Permissions to create pods and secrets in the Kubernetes cluster
- Helm version v3.2.0 or later
- Aws Account with Permission to create the necessary resources (efs, s3 and any other)
- GCP Account with Permission to create the necessary resources (Storage)
- Authentication keys for both EKS and GCP

## Quick Setup

### Install Crossplane

#### Install the Crossplane Helm chart

```
helm repo add \
crossplane-stable https://charts.crossplane.io/stable \
helm repo update
```

> Optional: Run the Helm dry-run to see all the Crossplane components Helm installs.

```
helm install crossplane \
crossplane-stable/crossplane \
--dry-run --debug \
--namespace crossplane-system \
--create-namespace
```

Install the Crossplane components using helm install.

```
helm install crossplane \
crossplane-stable/crossplane \
--namespace crossplane-system \
--create-namespace
```

> Verify: `kubectl get pods -n crossplane-system` and `kubectl api-resources | grep crossplane`

##### Install AWS provider(EFS)

`kubectl apply -f ./providers/provider-aws-efs.yaml`

> Verify: `kubectl get providers`

#### Create a Kubernetes secret for Aws

The provider requires credentials to create and manage AWS resources.
Providers use a Kubernetes Secret to connect the credentials to the provider.

Generate a Kubernetes Secret from your AWS key-pair and then configure the Provider to use it.

#### Generate an AWS key-pair file

```
[default]
aws_access_key_id = <>
aws_secret_access_key = <>
```

save the above as `aws-credentials.txt`

##### Create a Kubernetes secret with the AWS credentials

```
kubectl create secret \
generic aws-secret \
-n crossplane-system \
--from-file=creds=./aws-credentials.txt
```

> Verify: `kubectl describe secret aws-secret -n crossplane-system`

#### Create a ProviderConfig

A ProviderConfig customizes the settings of the AWS Provider.

Apply the ProviderConfig with this Kubernetes configuration file:

```
cat <<EOF | kubectl apply -f -
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-secret
      key: creds
EOF
```

### Create a managed resources (EFS)

##### Create filesystem

`kubectl apply -f ./resources/aws-efs/filesystem.yaml`

##### Create Access Point

`kubectl apply -f ./resources/aws-efs/accesspoint.yaml`

##### Create file system policy

`kubectl apply -f ./resources/aws-efs/filesystem-policy.yaml`

##### Create Mount Target

`kubectl apply -f ./resources/aws-efs/mount-target.yaml`

##### Create replication configuration

`kubectl apply -f ./resources/aws-efs/replication-configuration.yaml`

NB:
This is just the basic use/learning demo purposes. For production setup, there will be need to consider things `Compositions` `definitions` `Packages` and `Configurations`

## GCP

#### Create a Kubernetes secret for GCP

The provider requires credentials to create and manage GCP resources. Providers use a Kubernetes Secret to connect the credentials to the provider.

First, generate a Kubernetes Secret from a Google Cloud service account JSON file and then configure the Provider to use it.

##### Create a Kubernetes secret with the GCP credentials

```
kubectl create secret \
generic gcp-secret \
-n crossplane-system \
--from-file=creds=./gcp-credentials.json
```

> Verify: `kubectl describe secret gcp-secret -n crossplane-system`

#### Install Gcp Storage Provider

```
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-gcp-storage
spec:
  package: xpkg.upbound.io/upbound/provider-gcp-storage:v0.35.0
```

or
`kubectl apply -f ./resources/gcp/gcp-storage-provider.yaml`

#### Create a ProviderConfig:

```
apiVersion: gcp.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  projectID: abiding-truth-404513
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: gcp-secret
      key: creds
```

or
`kubectl apply -f ./resources/gcp/gcp-provider-config.yaml`

#### Create a managed resource

Create the Bucket with the following command:
`kubectl apply -f ./resources/gcp/gcp-storage.yaml`

or

```
apiVersion: storage.gcp.upbound.io/v1beta1
kind: Bucket
metadata:
  annotations:
    meta.upbound.io/example-id: storage/v1beta1/notification
  labels:
    testing.upbound.io/example-name: bucket
  name: bucket-demo-devops-nairobi-05
spec:
  forProvider:
    location: US
```

# Refs:

- https://docs.crossplane.io/latest/getting-started/provider-gcp/#create-a-kubernetes-secret-with-the-gcp-credentials
- https://docs.crossplane.io/latest/getting-started/provider-aws/
- https://marketplace.upbound.io/providers

# Next Steps

- Use Crossplane Compositions and XRDs to provision resources
- User Packages and Configurations to bundle together related resources
- Build a pipeline to create a OCI packaged and push to the registry
- Use the OCI packages and provision the resource through Crospplane Claim
