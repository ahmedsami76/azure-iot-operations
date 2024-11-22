### Step 1: Install `k3s`
Ensure your system is up-to-date.
```bash
curl -sfL https://get.k3s.io | sh -
```

Change the cluster name.
```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```
Edit the `~/.kube/config` file (replacing the "default" name to a new cluster name) as below:
```
...
clusters:
- cluster:
    certificate-authority-data: xxx
    server: https://127.0.0.1:6443
  name: aiok3scluster
contexts:
- context:
    cluster: aiok3scluster
    user: default
  name: aiok3scluster
current-context: aiok3scluster
...
```
Make sure the current context is set to the new cluster name
```bash
kubectl config use-context aiok3scluster
kubectl config current-context
```
```bash
sudo chmod 644 /etc/rancher/k3s/k3s.yaml
```


### Step 2: Install `helm`

For a scripted installation:
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
sudo bash get_helm.sh

```

### Step 2: Install `az`

For a scripted installation:
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

