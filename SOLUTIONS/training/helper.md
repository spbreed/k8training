Delete:
Shift -> D delete line
D -> g delete after cursor

:g/^$/d -> empty lines

grep -A 5

paste:
:set paste


tmux
 ctl + b -> " horizonal
 ctl + b -> % vertical
 ctl + b -> up 

 Kube token:

 kubeadm token create
 kubectl cluster-info

 kubeadm join  
 kubeadm join --token 24aabc.1ddacf4a18783cfc  172.31.44.80:6443 --discovery-token-unsafe-skip-ca-verification

 /var/nfs/general      172.31.44.80(rw,sync,no_subtree_check) 172.31.34.74(rw,sync,no_subtree_check) 172.31.43.138(rw,sync,no_subtree_check) (rw,sync,no_subtree_check)

## Storage
sudo apt update
sudo apt upgrade -y
Make a note of the internal IP addresses your four Kubernetes cluster are assigned.
sudo apt install nfs-kernel-server -y
Create a directory: sudo mkdir /var/nfs/general -p
Set correct owner and group on the directory: sudo chown nobody:nogroup /var/nfs/general
Edit the file /etc/exports such that the following line is added:
/var/nfs/general      x.x.x.x(rw,sync,no_subtree_check) y.y.y.y(rw,sync,no_subtree_check) z.z.z.z(rw,sync,no_subtree_check) a.a.a.a(rw,sync,no_subtree_check)

Where x.x.x.x, y.y.y.y, z.z.z.z, and a.a.a.a are the internal IP addresses of your Kubernetes nodes.
Restart the NFS server with the command sudo systemctl restart nfs-kernel-server.
2. Log in to each of your Kubernetes nodes, including the master, and execute the command sudo apt install nfs-common

#Taints and Tolerations:
This Pod can be scheduled on a node that has the taint 
dedicated=experimental:NoSchedule:

key,operator,value,effect

tolerations:
- key: dedicated
  operator: Equal
  value: experimental
  effect: NoSchedule
  
1) Get pods in a service
2) troubleshoor services
3) troubleshoot pods
4) dns services and pods nslookups
5) etcd 
