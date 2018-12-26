# k8training

wget https://raw.githubusercontent.com/spbreed/k8training/master/SOLUTIONS/training/ds.yaml

# Shell
### Delete:
- Shift -> D delete line
- D -> g delete after cursor
- :g/^$/d -> empty lines

- grep -A 5

## paste:
:set paste

### tmux
- ctl + b -> " horizonal
- ctl + b -> % vertical
- ctl + b -> up 

## Kubernetes 
 
 Kube token:

 - kubeadm token create
 - kubectl cluster-info
 - kubeadm join --token 24aabc.1ddacf4a18783cfc  172.31.44.80:6443 --discovery-token-unsafe-skip-ca-verification

## Storage
- sudo apt update
- sudo apt upgrade -y
- Make a note of the internal IP addresses your four Kubernetes cluster are assigned.
- sudo apt install nfs-kernel-server -y
- Create a directory: sudo mkdir /var/nfs/general -p
- Set correct owner and group on the directory: sudo chown nobody:nogroup /var/nfs/general
- Edit the file /etc/exports such that the following line is added:
- /var/nfs/general      x.x.x.x(rw,sync,no_subtree_check) y.y.y.y(rw,sync,no_subtree_check) z.z.z.z(rw,sync,no_subtree_check) a.a.a.a(rw,sync,no_subtree_check)
- /var/nfs/general      172.31.44.80(rw,sync,no_subtree_check) 172.31.34.74(rw,sync,no_subtree_check) 172.31.43.138(rw,sync,no_subtree_check) (rw,sync,no_subtree_check)
- Restart the NFS server with the command sudo systemctl restart nfs-kernel-server.
- Log in to each of your Kubernetes nodes, including the master, and execute the command sudo apt install nfs-common

## Persistent Volume
- create pv
- create pvc
- create volume
Attach to pod

## Resource quota
- Create NS
- Create quotas
- Create deployments

## Cluster IPS
- Cluster IP (default)
- Nodeport -> NodeIP port + ClusterIP
- Load balancer -> LB IP + NodeIP port + ClusterIP

## Deployments
- Replication Controllers
- Replicasets (Advanced selectors but no rolling updates) 
- Deployments

## Daemonsets
- Daemon sets bypasses K8 scheduler and do the scheduling based on the labels.Assign the labels and update the deployment

## SVC
```kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0 --port=8080```

## Taints and Tolerations:
This Pod can be scheduled on a node that has the taint 
- dedicated=experimental:NoSchedule:
- key,operator,value,effect

```
tolerations:
- key: dedicated
  operator: Equal
  value: experimental
  effect: NoSchedule
```

kubectl create namespace office

## Node upgrade:

- kubectl get nodes
- kubectl drain node2 --ignore-daemonsets
- apt-get update
- apt-get upgrade
- systemctl status kubelet 
- kubectl uncordon node2 

## Create User :

### Create key
openssl genrsa -out employee.key 2048
### Create CSR using key
openssl req -new -key employee.key -out employee.csr -subj "/CN=employee/O=bitnami"
### Create CRT using CSR and CA
openssl x509 -req -in employee.csr -CA CA_LOCATION/ca.crt -CAkey CA_LOCATION/ca.key -CAcreateserial -out employee.crt -days 500
### Create user with CRT & Key
kubectl config set-credentials employee --client-certificate=/home/employee/.certs/employee.crt  --client-key=/home/employee/.certs/employee.key
### Set context
kubectl config set-context employee-context --cluster=minikube --namespace=office --user=employee
kubectl --context=employee-context get pods

## Create Role
```
kind: Role
  apiVersion: rbac.authorization.k8s.io/v1beta1
  metadata:
    namespace: office
    name: deployment-manager
  rules:
  - apiGroups: ["", "extensions", "apps"]
    resources: ["deployments", "replicasets", "pods"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"] # You can also use ["*"]
```
## RoleBinding (Assign user to role)

```
kind: RoleBinding
  apiVersion: rbac.authorization.k8s.io/v1beta1
  metadata:
    name: deployment-manager-binding
    namespace: office
  subjects:
  - kind: User
    name: employee
    apiGroup: ""
  roleRef:
    kind: Role
    name: deployment-manager
    apiGroup: ""
```  
## Misc
### Get pods in a service
- kubectl get endpoints 
- kubectl get ep/svc-name -o json | jq '.subsets[].addresses[].targetRef.name'

### troubleshoot services
- Start service:
- systemctl | grep kub
- dpkg -l | grep docker

- kubeadm -init installs master node 
- starts a kubelet service "systemctl | grep kubelet-service"
- ps -aux | grep kubelet
- kubelet service watches /etc/kubernetes/manifests folder
  - kube-apiserver.yaml (for API service)
  - kube-controller-manager.yaml (controller)
  - kube-scheduler.yaml (scheduler service)

- kubectl get pods -n kube-system
- ps aux | grep kube-sc

- kubeadm -join installs worker node
  - kube-proxy
- ps aux | grep kube-proxy
- delete and create kube-proxy pod
- check iptables

3) troubleshoot pods
- kubectl run (creates a depolyment)
- kubectl exec (executes something)
- kubectl run busybox --image=busybox --command sleep "3600" 
- kubectl run -it --rm --restart=Never busybox --image=busybox -- sh
- kubectl exec -it busybox -- nslookup nginx-svc

svc -> endpoint -> pods -> proxy

4) dns services and pods nslookups
kubedns -> stub domains -> upstream servers

5) etcd 
6) static pods, system ctl pods
7) init containers
8) Rollout history
