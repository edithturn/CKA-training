# Working with custom columns

## Solution

```bash
kubectl get deployments.apps -n admin2406 \
  --sort-by=.metadata.name \
  -o custom-columns='DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[*].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace' \
  > /opt/admin2406_data
```

### Command explanation

- `get deployments.apps`: lists Deployment resources from the `apps` API group.
- `-n admin2406`: selects the `admin2406` namespace.
- `--sort-by=.metadata.name`: sorts the Deployments alphabetically by name.
- `-o custom-columns`: selects and names the required output columns.
- `.metadata.name`: Deployment name.
- `.spec.template.spec.containers[*].image`: container image or images.
- `.status.readyReplicas`: number of ready replicas.
- `.metadata.namespace`: namespace containing the Deployment.
- `> /opt/admin2406_data`: writes the output to the required file.

### Verification

```bash
cat /opt/admin2406_data
```

Result:

```text
DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE
deploy1      nginx             1                admin2406
deploy2      nginx:alpine      1                admin2406
deploy3      nginx:1.16        1                admin2406
deploy4      nginx:1.17        1                admin2406
deploy5      nginx:latest      1                admin2406
```


# Troubleshoot and Repair a Kubeconfig


### 1. Test the kubeconfig

Use `--kubeconfig` to ensure that `kubectl` tests the specified file instead of the default kubeconfig:

```bash
kubectl --kubeconfig=/root/CKA/admin.kubeconfig get nodes
```
Compare the broken kubeconfig with the working administrator configuration:

```bash
kubectl config view \
  --kubeconfig=/etc/kubernetes/admin.conf \
  --minify \
  -o jsonpath='{.clusters[0].cluster.server}{"\n"}'
```

Result:

```text
https://controlplane:6443
```

Change it!


```bash
kubectl --kubeconfig=/root/CKA/admin.kubeconfig get nodes
```

Successful result:

```text
NAME           STATUS   ROLES           VERSION
controlplane   Ready    control-plane   v1.35.0
node01         Ready    <none>          v1.35.0
```

### Troubleshooting logic

```text
Connection refused
    ↓
Inspect the configured API server endpoint
    ↓
Compare it with a working kubeconfig
    ↓
Find the incorrect port: 4380
    ↓
Change it to the correct port: 6443
    ↓
Test the repaired kubeconfig
```

A successful `kubectl get nodes` confirms that:

- The API server endpoint is correct.
- The certificate authority is valid.
- The user credentials are valid.
- The selected context references a valid cluster and user.
- The cluster can be accessed using the repaired kubeconfig.




# Create containers

```bash

cat > /tmp/mc-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: mc-pod
  namespace: mc-namespace
spec:
  containers:
  - name: mc-pod-1
    image: nginx:1-alpine
    env:
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName

  - name: mc-pod-2
    image: busybox:1
    command: ["sh", "-c", "while true; do date >> /var/log/shared/date.log; sleep 1; done"]
    volumeMounts:
    - name: shared
      mountPath: /var/log/shared

  - name: mc-pod-3
    image: busybox:1
    command: ["sh", "-c", "while [ ! -f /var/log/shared/date.log ]; do sleep 1; done; tail -n +1 -f /var/log/shared/date.log"]
    volumeMounts:
    - name: shared
      mountPath: /var/log/shared

  volumes:
  - name: shared
    emptyDir: {}
EOF
```


# Prepare node for install cri

```bash
ssh bob@node01
sudo apt install -y /root/cri-docker_0.3.16.3-0.debian.deb
sudo systemctl enable --now cri-docker.service
dpkg-query -W -f='${Status}\n' cri-dockerd
systemctl is-active cri-docker.service
systemctl is-enabled cri-docker.service
```


# CRD

```bash
kubectl get crd -o custom-columns=NAME:.metadata.name --no-headers | grep -i verticalpodautoscaler > /root/vpa-crds.txt
cat /root/vpa-crds.txt
```


# Create a Service with imperative commands

```bash
kubectl expose pod messaging --name=messaging-service --port=6379 --target-port=6379 --type=ClusterIP -n default
```


# Expose a deployment
```bash
kubectl expose deployment hr-web-app \
  --name=hr-web-app-service \
  --type=NodePort \
  --port=8080 \
  --target-port=8080
```



# Create a PV

```bash
cat <<EOF > pv-analytics.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/data-analytics
EOF
```


