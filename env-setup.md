# Setup K8S CLUSTER

## Minikube

> Using Ubuntu distro

```
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## Install Docker

```
sudo apt-get update \
sudo apt-get install ca-certificates curl gnupg \
sudo install -m 0755 -d /etc/apt/keyrings \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg \
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

# Add the repository to Apt sources:

```
echo   "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null \
sudo apt-get update \
sudo docker run hello-world 
```
`sudo usermod -aG docker $USER && newgrp docker`

`minikube start --force`

### Install kubectl

```
snap install kubectl --classic \
kubectl config get-contexts
```

### Install Helm

```
wget https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz \
tar -xvf  helm-v3.12.0-linux-amd64.tar.gz \
sudo mv linux-amd64  /usr/local/bin \
helm version
```

OR

### Helm using snap

`sudo snap install helm --classic`
