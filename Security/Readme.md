# Notes
## Security Primitives
kube-apiserver, is the center of all operations inside Kuberentes.
Who can access to the cluster?
* User IS
Files
Certificates
External Authentication providers
Services Accounts

What can they do?
* RBAC Authorization
* ABAC Authotization

Communication between ETCD Cluster, Kubelet, Kube Controller Manager, Kube Scheduler, Kube Proxy are secury using TLS Encryption.

## Authentication

Admins and Developers send requests to kube-apiserver, and it authenticates the requests before processing it.
Authentication happens using:
* Static Password File
* Static Token File
* Certificates
* Identity Services

