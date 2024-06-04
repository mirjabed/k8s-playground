# Getting started with k8s


## Install Docker

[Official documentation][def]

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo groupadd docker
sudo usermod -aG docker $USER

docker run hello-world
```

## Install Kubectl
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly

sudo apt-get update
sudo apt-get install -y kubectl
```

## Install Helm
Official installation [guide][def4]

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

## Install go
```bash
sudo snap update
sudo snap install go --classic
```

## Install Kind
[kind][def2]
```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create k8s cluster
# kind create cluster --name dev --config cluster.yaml 
kind create cluster --name dev --config - << EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: "192.168.178.114" # <- IP of the VM
  apiServerPort: 6443
nodes:
- role: control-plane
  image: kindest/node:v1.28.9
- role: worker
  image: kindest/node:v1.28.9
- role: worker
  image: kindest/node:v1.28.9
- role: worker
  image: kindest/node:v1.28.9
EOF

# Check cluster info
kubectl cluster-info --context kind-dev
```


## MetalLB

[MetalLB Installation doc][def3]
```bash
# see what changes would be made, returns nonzero returncode if different
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# actually apply the changes, returns nonzero returncode on errors only
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system

```

### Installation With Helm
You can install MetalLB with Helm by using the Helm chart repository: https://metallb.github.io/metallb
```bash
helm repo add metallb https://metallb.github.io/metallb
helm upgrade --install --create-namespace --namespace metallb-system metallb metallb/metallb

kubectl apply -f - << EOF
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ip-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.19.2.1-172.19.2.10 # <- Use IP from docker bridge network named kind
EOF
kubectl apply -f - << EOF
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2advertisement
  namespace: metallb-system
spec:
  ipAddressPools:
  - ip-pool
EOF

```

## Nginx Controller

Install the CRD
```bash
kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v3.5.1/deploy/crds.yaml
```

Install controller
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm upgrade --install --create-namespace --namespace nginx-controller nginx-controller ingress-nginx/ingress-nginx
```

```bash
helm upgrade --install --create-namespace --namespace demo password-generato .
```

[def]: https://docs.docker.com/engine/install/ubuntu/
[def2]: https://kind.sigs.k8s.io/docs/user/quick-start#installing-from-release-binaries
[def3]: https://metallb.universe.tf/installation/
[def4]: https://helm.sh/docs/intro/install/#from-apt-debianubuntu
