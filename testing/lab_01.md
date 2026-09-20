# Kubernetes Upgrade: v1.34.0 to v1.35.0

## 1. Check the initial state

```bash
kubectl get nodes
kubectl get pods -o wide
```

Confirm that:

- Both nodes are running Kubernetes `v1.34.0`.
- The `gold-nginx` pod is healthy.
- The pod is running on `node01`.

## 2. Drain the control-plane node

```bash
kubectl drain controlplane --ignore-daemonsets --delete-emptydir-data
```

Draining a node:

- Prevents new pods from being scheduled on it.
- Evicts regular workload pods.
- Automatically cordons the node.
- Ignores pods managed by DaemonSets.

Verify:

```bash
kubectl get nodes
kubectl get pods -o wide
```

Expected:

```text
controlplane   Ready,SchedulingDisabled
```

The `gold-nginx` pod should be running on `node01`.

## 3. Configure the Kubernetes v1.35 repository

Check the current repository:

```bash
grep -R "pkgs.k8s.io" /etc/apt/sources.list.d/
```

Change the repository from `v1.34` to `v1.35`:

```bash
sudo sed -i 's|core:/stable:/v1.34/deb/|core:/stable:/v1.35/deb/|' \
  /etc/apt/sources.list.d/kubernetes.list
```

Update the package index:

```bash
sudo apt-get update
```

Find the exact `1.35.0` package version:

```bash
apt-cache madison kubeadm | grep '1.35.0'
```

Example result:

```text
kubeadm | 1.35.0-1.1 | ...
```

## 4. Upgrade kubeadm on the control plane

```bash
sudo apt-mark unhold kubeadm
sudo apt-get install kubeadm=1.35.0-1.1
sudo apt-mark hold kubeadm
```

Verify:

```bash
dpkg-query -W kubeadm
kubeadm version -o short
```

Expected:

```text
kubeadm  1.35.0-1.1
v1.35.0
```

This only upgrades the `kubeadm` utility. It does not upgrade the cluster itself.

## 5. Upgrade the control-plane components

Review the proposed upgrade:

```bash
sudo kubeadm upgrade plan
```

Apply the upgrade:

```bash
sudo kubeadm upgrade apply v1.35.0
```

Enter `y` when prompted.

Expected message:

```text
SUCCESS! A control plane node of your cluster was upgraded to "v1.35.0".
```

Verify the API server:

```bash
kubectl version
```

Expected at this stage:

```text
Client Version: v1.34.x
Server Version: v1.35.0
```

Optionally, verify the API server image:

```bash
kubectl get pod kube-apiserver-controlplane -n kube-system \
  -o jsonpath='{.spec.containers[0].image}{"\n"}'
```

Expected:

```text
registry.k8s.io/kube-apiserver:v1.35.0
```

## 6. Upgrade kubelet and kubectl on the control plane

```bash
sudo apt-mark unhold kubelet kubectl
sudo apt-get install kubelet=1.35.0-1.1 kubectl=1.35.0-1.1
sudo apt-mark hold kubelet kubectl
```

Verify the packages and binaries:

```bash
dpkg-query -W kubelet kubectl
kubelet --version
kubectl version --client
```

Expected versions:

```text
kubelet  1.35.0-1.1
kubectl  1.35.0-1.1
Kubernetes v1.35.0
Client Version: v1.35.0
```

Restart the kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
sudo systemctl status kubelet --no-pager
```

Verify the node version:

```bash
kubectl get nodes
```

Expected:

```text
controlplane   Ready,SchedulingDisabled   control-plane   ...   v1.35.0
node01         Ready                      <none>          ...   v1.34.0
```

## 7. Uncordon the control plane

```bash
kubectl uncordon controlplane
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
controlplane   Ready   control-plane   ...   v1.35.0
node01         Ready   <none>          ...   v1.34.0
```

The control plane must be schedulable before draining `node01`.

## 8. Drain node01

```bash
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data
```

This evicts `gold-nginx` from `node01`. The Deployment creates a replacement pod on `controlplane`.

Verify:

```bash
kubectl get nodes
kubectl get pods -o wide
```

Expected:

- `node01` shows `Ready,SchedulingDisabled`.
- `gold-nginx` runs on `controlplane`.

The pod may temporarily show:

```text
0/1   ContainerCreating
```

Watch it until it becomes ready:

```bash
kubectl get pods -o wide -w
```

Press `Ctrl+C` when it reaches `1/1 Running`.

## 9. Connect to node01

```bash
ssh node01
hostname
```

Expected hostname:

```text
node01
```

## 10. Configure the Kubernetes v1.35 repository on node01

```bash
grep -R "pkgs.k8s.io" /etc/apt/sources.list.d/
```

Change the repository if it still references `v1.34`:

```bash
sudo sed -i 's|core:/stable:/v1.34/deb/|core:/stable:/v1.35/deb/|' \
  /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

Confirm the required version:

```bash
apt-cache madison kubeadm | grep '1.35.0'
```

## 11. Upgrade kubeadm on node01

```bash
sudo apt-mark unhold kubeadm
sudo apt-get install kubeadm=1.35.0-1.1
sudo apt-mark hold kubeadm
```

Verify:

```bash
dpkg-query -W kubeadm
kubeadm version -o short
```

Expected:

```text
kubeadm  1.35.0-1.1
v1.35.0
```

## 12. Upgrade the worker-node configuration

```bash
sudo kubeadm upgrade node
```

The commands differ by node type:

- First control-plane node: `kubeadm upgrade apply v1.35.0`
- Worker node: `kubeadm upgrade node`

## 13. Upgrade kubelet and kubectl on node01

```bash
sudo apt-mark unhold kubelet kubectl
sudo apt-get install kubelet=1.35.0-1.1 kubectl=1.35.0-1.1
sudo apt-mark hold kubelet kubectl
```

Verify:

```bash
dpkg-query -W kubelet kubectl
kubelet --version
kubectl version --client
```

Restart the kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
sudo systemctl status kubelet --no-pager
```

Return to the control-plane node:

```bash
exit
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
controlplane   Ready                      control-plane   ...   v1.35.0
node01         Ready,SchedulingDisabled   <none>          ...   v1.35.0
```

## 14. Uncordon node01

```bash
kubectl uncordon node01
```

This makes the upgraded worker available for scheduling again.

## 15. Final verification

```bash
kubectl get nodes
kubectl get pods -o wide
kubectl get deployment gold-nginx
kubectl version
```

Confirm that:

- Both nodes are `Ready`.
- Both nodes are running `v1.35.0`.
- Neither node shows `SchedulingDisabled`.
- `gold-nginx` is `1/1 Running`.
- `gold-nginx` is running on `controlplane`.
- The client and server versions are `v1.35.0`.

## Important reminders

- `kubectl drain` automatically cordons a node.
- `kubectl uncordon` makes the node schedulable again.
- `kubeadm version` only verifies the `kubeadm` binary.
- `kubectl version` verifies the client and API server.
- `kubectl get nodes` reports each node's kubelet version.
- Always upgrade and verify one node before continuing to the next.
- Use `apt-cache madison` to confirm the exact package revision instead of assuming it is `1.35.0-1.1`.