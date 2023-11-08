# Crossplane-iac-Nairobi-DevOps-Community-

A simple demo on Using crossplane to create infrastructure for the Nairobi DevOps Community

# Prereliquisites

[-] A Kubernetes cluster with at least 2 GB of RAM (Consider Minikube/Kind)
[-] Permissions to create pods and secrets in the Kubernetes cluster
[-] Helm version v3.2.0 or later
[-] Aws Account with Permission to create the necessary resources (efs, s3 and any other)
[-] GCP Account with Permission to create the necessary resources (Storage)
[-] Authentication keys for both EKS and GCP

## Quick Setup

### Install Crossplane

#### Install the Crossplane Helm chart

```
helm repo add \
crossplane-stable https://charts.crossplane.io/stable
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

> Verify: `kubectl get pods` and `kubectl api-resources | grep crossplane.`

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

Apply the ProviderConfig with the this Kubernetes configuration file:

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
