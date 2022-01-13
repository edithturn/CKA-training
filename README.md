
#       :octopus: **Certified Kubernetes Administrator**  :dolphin:

This repository contains definitions, tips, sources, and many commands for practice and that I am currently using to prepare for my Certified Kubernetes Administrator exam.

# `` REPOSITORY IN PROGRESS ... `` :carousel_horse: :raising_hand:  :tractor:

```bash
cat ~/.kube/config | grep current | awk '{print $2}'
k config get-contexts -o name > /tmp/contexts

```
kubectl 

```

Manage applications
* Statefullset
kubectl 

```bash
kubectl get pods -n kube-system
```

ETCDCTL is the CLI tool used to interact with ETCD.


## Commands

```bash
kubectl taint node worker not=used:NoSchedule

ssh master
cat /etc/hosts

ls /etc/kubernetes/manifests/

systemctl status kubelet.service
systemctl restart kubelet.service

kubectl describe nodes master100 | grep -i taint
kubectl get pod -A

kubectl auth can-i create pods
kubectl auth can-i create secrets
kubectl auth can-i create configMaps
kubectl api-resoirces -o 

Security <-
```

## Switch Namespaces

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev

kubectl config set-context $(kubectl config current-context) --namespace=prod

```

## Command

k get ns | wc -l
k get ns -n research




# Resources:

https://github.com/ahmetb/kubernetes-network-policy-recipes

