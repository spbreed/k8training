## Contents


- 1 Introduction
   - 1.1 Labs
- 2 Basics of Kubernetes
   - 2.1 Labs
- 3 Installation and Configuration
   - 3.1 Labs
- 4 Kubernetes Architecture
   - 4.1 Labs
- 5 APIs and Access
   - 5.1 Labs
- 6 API Objects
   - 6.1 Labs
- 7 Managing State With Deployments
   - 7.1 Labs
- 8 Services
   - 8.1 Labs
- 9 Volumes and Data
   - 9.1 Labs
- 10 Ingress
   - 10.1 Labs
- 11 Scheduling
   - 11.1 Labs
- 12 Logging and Troubleshooting
   - 12.1 Labs
- 13 Custom Resource Definition
   - 13.1 Labs
- 14 Kubernetes Federation
   - 14.1 Labs iv CONTENTS
- 15 Helm
   - 15.1 Labs
- 16 Security
   - 16.1 Labs


**List of Figures**

```
3.1 External Access via Browser............................................ 18
```
```
v
```

vi LIST OF FIGURES


**Chapter 1**

**Introduction**

### 1.1 Labs

### Exercise 1.1: Configuring the System for sudo

It is very dangerous to run a **root shell** unless absolutely necessary: a single typo or other mistake can cause serious (even
fatal) damage.

Thus, the sensible procedure is to configure things such that single commands may be run with superuser privilege, by using
the **sudo** mechanism. With **sudo** the user only needs to know their own password and never needs to know the root password.

If you are using a distribution such as **Ubuntu** , you may not need to do this lab to get **sudo** configured properly for the course.
However, you should still make sure you understand the procedure.

To check if your system is already configured to let the user account you are using run **sudo** , just do a simple command like:

$ sudo ls

You should be prompted for your user password and then the command should execute. If instead, you get an error message
you need to execute the following procedure.

Launch a root shell by typing **su** and then giving the **root** password, not your user password.

On all recent **Linux** distributions you should navigate to the/etc/sudoers.dsubdirectory and create a file, usually with the
name of the user to whom root wishes to grant **sudo** access. However, this convention is not actually necessary as **sudo** will
scan all files in this directory as needed. The file can simply contain:

student ALL=(ALL) ALL

if the user isstudent.

An older practice (which certainly still works) is to add such a line at the end of the file/etc/sudoers. It is best to do so using
the **visudo** program, which is careful about making sure you use the right syntax in your edit.

You probably also need to set proper permissions on the file by typing:

$ chmod 440 /etc/sudoers.d/student

#### 1


#### 2 CHAPTER 1. INTRODUCTION

(Note some **Linux** distributions may require 400 instead of 440 for the permissions.)

After you have done these steps, exit the root shell by typingexitand then try to dosudo lsagain.

There are many other ways an administrator can configure **sudo** , including specifying only certain permissions for certain
users, limiting searched paths etc. The/etc/sudoersfile is very well self-documented.

However, there is one more setting we highly recommend you do, even if your system already has **sudo** configured. Most
distributions establish a different path for finding executables for normal users as compared to root users. In particular the
directories/sbinand/usr/sbinare not searched, since **sudo** inherits thePATHof the user, not the full root user.

Thus, in this course we would have to be constantly reminding you of the full path to many system administration utilities;
any enhancement to security is probably not worth the extra typing and figuring out which directories these programs are in.
Consequently, we suggest you add the following line to the.bashrcfile in your home directory:

PATH=$PATH:/usr/sbin:/sbin

If you log out and then log in again (you don’t have to reboot) this will be fully effective.


**Chapter 2**

**Basics of Kubernetes**

### 2.1 Labs

### Exercise 2.1: View Online Resources

### Visit kubernetes.io

With such a fast changing project, it is important to keep track of updates. The main place to find documentation of the current
version ishttps://kubernetes.io/.

1. Open a browser and visit thehttps://kubernetes.io/website.
2. In the upper right hand corner, use the drop down to view the versions available. It will say something likev1.12.
3. Select the top level link forDocumentation. The links on the left of the page can be helpful in navigation.
4. As time permits navigate around other sub-pages such asSETUP,CONCEPTS, andTASKSto become familiar with the
    layout.

### Track Kubernetes Issues

There are hundreds, perhaps thousands, working on Kubernetes every day. With that many people working in parallel there
are good resources to see if others are experiencing a similar outage. Both the source code as well as feature and issue
tracking are currently ongithub.com.

1. To view the main page use your browser to visithttps://github.com/kubernetes/kubernetes/
2. Click on various sub-directories and view the basic information available.
3. Update your URL to point tohttps://github.com/kubernetes/kubernetes/issues. You should see a series of
    issues, feature requests, and support communication.

#### 3


#### 4 CHAPTER 2. BASICS OF KUBERNETES

4. In the search box you probably see some existing text likeis:issue is:openwhich allows you to filter on the kind of
    information you would like to see. Append the search string to read:is:issue is:open label:kind/bugthen press
    enter.
5. You should now see bugs in descending date order. Across the top of the issues a menu area allows you to view entries
    by author, labels, projects, milestones, and assignee as well. Take a moment to view the various other selection criteria.
6. Some times you may want to exclude a kind of output. Update the URL again, but precede the label with a minus sign,
    like:is:issue is:open -label:kind/bug. Now you see everything except bug reports.
7. Explore the page with the remaining time left.


**Chapter 3**

**Installation and Configuration**

### 3.1 Labs

### Exercise 3.1: Install Kubernetes

### Overview

There are several Kubernetes installation tools provided by various vendors. In this lab we will learn to use **kubeadm**. As a
community-supported independent tool, it is planned to become the primary manner to build a Kubernetes cluster.

The labs were written using **Ubuntu** instances running on **G** oogle **C** loud **P** latform ( **GCP** ). They have been written to be
vendor-agnostic so could run on AWS, local hardware, or inside of virtualization to give you the most flexibility and options.
Each platform will have different access methods and considerations. As of v1.12.1 the minimum (as in barely works) size for
**VirtualBox** is 3vCPU/1G memory/5G minimal OS formasterand 1vCPU/1G memory/5G minimal OS for worker node.

If using your own equipment you will have to disable swap on every node. There may be other requirements which will be
shown as warnings or errors when using the **kubeadm** command. While most commands are run as a regular user, there are
some which require root privilege. Please configure **sudo** access as shown in a previous lab. You If you are accessing the
nodes remotely, such as with **GCP** or **AWS** , you will need to use an SSH client such as a local terminal or **PuTTY** if not using
**Linux** or a Mac. You can download **PuTTY** fromwww.putty.org. You would also require a.pemor.ppkfile to access the
nodes. Each cloud provider will have a process to download or create this file. If attending in-person instructor led training the
file will be made available during class.

In the following exercise we will install Kubernetes on a single node then grow the cluster, adding more compute resources.
Both nodes used are the same size, providing 2 vCPUs and 7.5G of memory. Smaller nodes could be used, but would run
slower.

Various exercises will use YAML files, which are included in the text. You are encouraged to write the files when possible, as
the syntax of YAML has white space indentation requirements that are important to learn. An important note, **do not** use tabs
in your YAML files, **white space only. Indentation matters.**

If using a PDF the use of copy and paste often does not paste the single quote correctly. It pastes as a back-quote instead.
You will need to modify it by hand. The files have also been made available as a compressed **tar** file. You can view the
resources by navigating to this URL:

#### 5


#### 6 CHAPTER 3. INSTALLATION AND CONFIGURATION

https://training.linuxfoundation.org/cm/LFS

To login use user:LFtrainingand a password of:Penguin

Once you find the name and link of the current file, which will change as the course updates, use **wget** to download the file
into your node from the command line then expand it like this:

$ wget https://training.linuxfoundation.org/cm/LFS258/LFS258V2018-11-13SOLUTIONS.tar.bz2\
--user=LFtraining --password=Penguin

$ tar -xvf LFS258V2018-11-13SOLUTIONS.tar.bz

(Note: depending on your software, if you are cutting and pasting the above instructions, the underscores may disappear and
be replaced by spaces, so you may have to edit the command line by hand!)

```
While Ubuntu 18 bionichas become the typical version to deploy the Kubernetes repository does not
yet have compatible binaries at the time of this writing. Whilexenialbinaries can be used there are
many additional steps necessary to complete the labs. Ubuntu 18 is expected to be available by the
time Kubernetes v.1.13 is released.
```
### Install Kubernetes

Log into your nodes. If attending in-person instructor led training the node IP addresses will be provided by the instructor. You
will need to use a.pemor.ppkkey for access, depending on if you are using **ssh** from a terminal or **PuTTY**. The instructor
will provide this to you.

1. Open a terminal session on your first node. For example, connect via **PuTTY** or **SSH** session to the first **GCP** node. The
    user name may be different than the one shown,student. The IP used in the example will be different than the one you
    will use.

```
[student@laptop ~]$ ssh -i LFS458.pem student@35.226.100.
The authenticity of host ’54.214.214.156 (35.226.100.87)’ can’t be established.
ECDSA key fingerprint is SHA256:IPvznbkx93/Wc+ACwXrCcDDgvBwmvEXC9vmYhk2Wo1E.
ECDSA key fingerprint is MD5:d8:c9:4b:b0:b0:82:d3:95:08:08:4a:74:1b:f6:e1:9f.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added ’35.226.100.87’ (ECDSA) to the list of known hosts.
<output_omitted>
```
2. Becomerootand update and upgrade the system. Answer any questions to use the defaults.

```
student@lfs458-node-1a0a:~$ sudo -i
```
```
root@lfs458-node-1a0a:~# apt-get update && apt-get upgrade -y
<output_omitted>
```
3. The main choices for a container environment are **Docker** and **cri-o**. We will user **Docker** for class, as **cri-o** requires a
    fair amount of extra work to enable for Kubernetes. As **cri-o** is open source the community seems to be heading towards
    its use.

```
root@lfs458-node-1a0a:~# apt-get install -y docker.io
<output-omitted>
```
4. Add new repo for kubernetes. You could also get a tar file or use code from GitHub. Create the file and add an entry for
    the main repo for your distribution. As we are still usingUbuntu 16.04add thekubernetes-xenialwith the key word
    main. Note there are four sections to the entry.

```
root@lfs458-node-1a0a:~# vim /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
```

#### 3.1. LABS 7

5. Add a GPG key for the packages. The command spans three lines. You can omit the backslash when you type. TheOK
    is the expected output, not part of the command.

```
root@lfs458-node-1a0a:~# curl -s \
https://packages.cloud.google.com/apt/doc/apt-key.gpg \
| apt-key add -
OK
```
6. Update with new repo, which will download new repo information.

```
root@lfs458-node-1a0a:~# apt-get update
<output-omitted>
```
7. Install the software. There are regular releases the newest of which can be used by omitting the equal sign and version
    information on the command line. Historically new version have lots of changes and a good chance of a bug or five.

```
root@lfs458-node-1a0a:~# apt-get install -y \
kubeadm=1.12.1-00 kubelet=1.12.1-00 kubectl=1.12.1-
<output-omitted>
```
8. Deciding which pod network to use for Container Networking Interface ( **CNI** ) should take into account the expected
    demands on the cluster. There can be only one pod network per cluster, although the **CNI-Genie** project is trying to
    change this.
    The network must allow container-to-container, pod-to-pod, pod-to-service, and external-to-service communications. As
    **Docker** uses host-private networking, using thedocker0virtual bridge andvethinterfaces would require being on that
    host to communicate.
    We will use **Calico** as a network plugin which will allow us to useNetwork Policieslater in the course. Currently
    **Calico** does not deploy using CNI by default. The 3.3 version of **Calico** has more than one configuration file for flexibility
    with RBAC. Download the configuration files for. Once downloaded look for the expected IPV4 range for containers to
    use.
    A short url for each file is shown, the longer URLs can be found here:https://docs.projectcalico.org/v3.3/
    getting-started/kubernetes/installation/hosted/rbac-kdd.yamland:https://docs.projectcalico.org/
    v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/
    calico.yaml

```
root@lfs458-node-1a0a:~# wget https://tinyurl.com/yb4xturm \
-O rbac-kdd.yaml
```
```
root@lfs459-node-1a0a:~# wget https://tinyurl.com/y8lvqc9g \
-O calico.yaml}
```
9. Use **less** to page through the file. Look for the IPV4 pool expected by the containers. There are many different config-
    uration settings in this file. Take a moment to view the entire file. TheCALICO_IPV4POOL_CIDRmust match the value
    given to **kubeadm init** in the following step, whatever the value may be.

```
root@lfs458-node-1a0a:~# less calico.yaml
....
# Configure the IP Pool from which Pod IPs will be chosen.
```
- name: CALICO_IPV4POOL_CIDR
    value: "192.168.0.0/16"
....
10. Initialize the master. Read through the output line by line. Expect the output to change as the software matures. At
the end are configuration directions to run as a non-root user. The token is mentioned as well. This information can be
found later with the **kubeadm token list** command. The output also directs you to create a pod network to the cluster,
which will be our next step. Pass the network settings **Calico** has in its configuration file, found in the previous step.
**Please note:** the output lists several commands which following commands will complete. Read the next step before
further typing.

```
root@lfs458-node-1a0a:~# kubeadm init --pod-network-cidr 192.168.0.0/
```

#### 8 CHAPTER 3. INSTALLATION AND CONFIGURATION

```
[init] using Kubernetes version: v1.12.
[preflight] running pre-flight checks
[preflight/images] Pulling images required for setting up a
Kubernetes cluster
[preflight/images] This might take a minute or two, depending
on the speed of your internet connection
```
```
<output-omitted>
```
```
Your Kubernetes master has initialized successfully!
```
```
To start using your cluster, you need to run the following as
a regular user:
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
```
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options
listed at:
https://kubernetes.io/docs/concepts/cluster-administration/addons/
```
```
You can now join any number of machines by running the following
on each node as root:
```
```
kubeadm join 10.128.0.4:6443 --token rdnhok.g8mb6lfgesunanvh
--discovery-token-ca-cert-hash
sha256:66350d154fc0169b5bb5fd50c04b72468195e356d78d95f137ed55e995402f
```
11. As suggested in the directions at the end of the previous output we will allow anon-rootuser admin level access to the
    cluster. Take a quick look at the configuration file once it has been copied and the permissions fixed.

```
root@lfs458-node-1a0a:~# exit
logout
```
```
student@lfs458-node-1a0a:~$ mkdir -p $HOME/.kube
```
```
student@lfs458-node-1a0a:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
```
student@lfs458-node-1a0a:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
```
student@lfs458-node-1a0a:~$ less .kube/config
apiVersion: v
clusters:
```
- cluster:
<output_omitted>
12. Apply the network plugin configuration to your cluster. Remember to copy the file to the current, non-root user directory
first.

```
student@lfs458-node-1a0a:~$ sudo cp /root/rbac-kdd.yaml.
```
```
student@lfs458-node-1a0a:~$ kubectl apply -f rbac-kdd.yaml
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
```
```
student@lfs458-node-1a0a:~$ sudo cp /root/calico.yaml.
```
```
student@lfs458-node-1a0a:~$ kubectl apply -f calico.yaml
configmap/calico-config created
service/calico-typha created
deployment.apps/calico-typha created
poddisruptionbudget.policy/calico-typha created
<output_omitted>
```

#### 3.1. LABS 9

13. While many objects have short names, a **kubectl** command can be a lot to type. We will enable **bash** auto-completion.
    Begin by adding the settings to the current shell. Then update the~/.bashrcfile to make it persistent.

```
student@lfs458-node-1a0a:~$ source <(kubectl completion bash)
```
```
student@lfs458-node-1a0a:~$ echo "source <(kubectl completion bash)" >> ~/.bashrc
```
14. Test by describing the node again. Type the first three letters of the sub-command then type the **Tab** key. Auto-completion
    assumes thedefaultnamespace. Pass the namespace first to use auto-completion with a different namespace. By
    pressing **Tab** multiple times you will see a list of possible values. Continue typing until a unique name is used.

```
student@lfs458-node-1a0a:~$ kubectl des<Tab> n<Tab><Tab> lfs458-<Tab>
```
```
student@lfs458-node-1a0a:~$ kubectl -n kube-s<Tab> g<Tab> po e<Tab>
```
### Exercise 3.2: Grow the Cluster

Open another terminal and connect into a your second node. Install **Docker** and Kubernetes software. These are the many,
but not all, of the steps we did on the master node.

The book will use the **lfs458-worker** prompt for the node being added to help keep track of the proper node for each command.
Note that the prompt indicates both the user and system upon which run the command.

1. Using the same process as before connect to a second node. If attending ILT use the same.pemkey and a new IP
    provided by the instructor to access the new node. Giving a title or color to the new terminal window is probably a good
    idea to keep track of the two systems. The prompts can look very similar.

```
student@lfs458-worker:~$ sudo -i
```
```
root@lfs458-worker:~# apt-get update && apt-get upgrade -y
```
```
root@lfs458-worker:~# apt-get install -y docker.io
```
```
root@lfs458-worker:~# vim /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
```
```
root@lfs458-worker:~# curl -s \
https://packages.cloud.google.com/apt/doc/apt-key.gpg \
| apt-key add -
```
```
root@lfs458-worker:~# apt-get update
```
```
root@lfs458-worker:~# apt-get install -y \
kubeadm=1.12.1-00 kubelet=1.12.1-00 kubectl=1.12.1-
```
2. Find the IP address of your master server. The interface name will be different depending on where the node is running.
    Currently inside of **GCE** the primary interface for this node type isens4. Your interfaces names may be different. From
    the output we know our master node IP is10.128.0.3.

```
student@lfs458-node-1a0a:~$ ip addr show ens4 | grep inet
inet 10.128.0.3/32 brd 10.128.0.3 scope global ens
inet6 fe80::4001:aff:fe8e:2/64 scope link
```
3. Find the token on the master node. The token lasts 24 hours by default. If it has been longer, and no token is present
    you can generate a new one with the **sudo kubeadm token create** command, seen in the following command.

```
student@lfs458-node-1a0a:~$ sudo kubeadm token list
TOKEN TTL EXPIRES USAGES DESCRIPTION
27eee4.6e66ff60318da929 23h 2017-11-03T13:27:33Z
authentication,signing The default bootstrap token generated
by ’kubeadm init’....
```

#### 10 CHAPTER 3. INSTALLATION AND CONFIGURATION

4. **Only if the token has expired** , you can create a new token, to use as part of the join command.

```
student@lfs458-node-1a0a:~$ sudo kubeadm token create
27eee4.6e66ff60318da
```
5. Starting in v1.9 you should create and use a Discovery Token CA Cert Hash created from the master to ensure the node
    joins the cluster in a secure manner. Run this on the master node or wherever you have a copy of the CA file. You will
    get a long string as output.

```
student@lfs458-node-1a0a:~$ openssl x509 -pubkey \
-in /etc/kubernetes/pki/ca.crt | openssl rsa \
-pubin -outform der 2>/dev/null | openssl dgst \
-sha256 -hex | sed ’s/^.* //’
6d541678b05652e1fa5d43908e75e67376e994c3483d6683f2a18673e5d2a1b
```
6. Use the token and hash, in this case assha256:<hash>to join the cluster from the second node. Use the **private** IP
    address of the master server and port 6443. The output of the **kubeadm init** on the master also has an example to use,
    should it still be available.

```
root@lfs458-worker:~# kubeadm join \
--token 27eee4.6e66ff60318da929 \
10.128.0.3:6443 \
--discovery-token-ca-cert-hash \
sha256:6d541678b05652e1fa5d43908e75e67376e994c3483d6683f2a18673e5d2a1b
```
```
[preflight] Running pre-flight checks.
[WARNING FileExisting-crictl]: crictl not found in system path
[discovery] Trying to connect to API Server "10.142.0.2:6443"
[discovery] Created cluster-info discovery client, requesting info from
"https://10.142.0.2:6443"
[discovery] Requesting info from "https://10.142.0.2:6443" again to
validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS
certificate validates against pinned roots, will
use API Server "10.142.0.2:6443"
[discovery] Successfully established connection with API Server
"10.142.0.2:6443"
This node has joined the cluster:
* Certificate signing request was sent to master and a response
was received.
* The Kubelet was informed of the new secure connection details.
Run ’kubectl get nodes’ on the master to see this node join the cluster.
```
7. Try to run the **kubectl** command on the secondary system. It should fail. You do not have the cluster or authentication
    keys in your local.kube/configfile.

```
root@lfs458-worker:~# exit
```
```
student@lfs458-worker:~$ kubectl get nodes
The connection to the server localhost:8080 was refused
```
- did you specify the right host or port?

```
student@lfs458-worker:~$ ls -l .kube
ls: cannot access ’.kube’: No such file or directory
```
### Exercise 3.3: Finish Cluster Setup

1. View the available nodes of the cluster. It can take a minute or two for the status to change fromNotReadytoReady. The
    NAMEfield can be used to look at the details. Your node name will be different. Note themasternode saysNotReady,
    which is due to ataint.


#### 3.1. LABS 11

```
student@lfs458-node-1a0a:~$ kubectl get node
NAME STATUS ROLES AGE VERSION
lfs458-node-1a0a NotReady master 18m v1.12.
lfs458-worker Ready <none> 3m25s v1.12.
```
2. Look at the details of the node. Work line by line to view the resources and their current status. Notice the status of
    Taints. The master wont allow non-internal pods by default for security reasons. Take a moment to read each line of
    output, some appear to be an error until you notice the status showsFalse.

```
student@lfs458-node-1a0a:~$ kubectl describe node lfs458-node-1a0a
Name: lfs458-node-1a0a
Roles: master
Labels: beta.kubernetes.io/arch=amd
beta.kubernetes.io/os=linux
kubernetes.io/hostname=lfs458-node-1a0a
node-role.kubernetes.io/master=
Annotations: kubeadm.alpha.kubernetes.io/cri-socket=/var/run/dockershim.sock
node.alpha.kubernetes.io/ttl=
volumes.kubernetes.io/controller-managed-attach-detach=true
CreationTimestamp: Sun, 29 Jul 2018 21:29:32 +
Taints: node-role.kubernetes.io/master:NoSchedule
<output_omitted>
```
3. Allow the master server to run non-infrastructure pods. The master node begins tainted for security and performance
    reasons. Will will allow usage of the node in the training environment, but this step may be skipped in a production
    environment. Note the minus sign (-) at the end, which is the syntax to remove a taint. As the second node does not
    have the taint you will get anot founderror.

```
student@lfs458-node-1a0a:~$ kubectl describe node | grep -i taint
Taints: node-role.kubernetes.io/master:NoSchedule
Taints: <none>
```
```
student@lfs458-node-1a0a:~$ kubectl taint nodes \
--all node-role.kubernetes.io/master-
```
```
node/lfs458-node-1a0a untainted
error: taint "node-role.kubernetes.io/master:" not found
```
4. Now that the master node is able to execute any pod we **may** find there is a new taint. This behavior began with v1.12.0,
    requiring a newly added node to be enabled. View then remove the taint if present. It can take a minute or two for the
    scheduler to deploy the remaining pods.

```
student@lfs458-node-1a0a:~$ kubectl describe node | grep -i taint
Taints: node.kubernetes.io/not-ready:NoSchedule
Taints: <none>
```
```
student@lfs458-node-1a0a:~$ kubectl taint nodes \
--all node.kubernetes.io/not-ready-
```
```
node/lfs58-node-1a0a untainted
error: taint "node.kubernetes.io/not-ready:" not found
```
5. Another ”undocumented feature” in v1.12.1 is that the taint removal does not always work the first time. Check to see
    if the taint has been removed. You may have to remove the taint two or three times before it is actually gone. Wait 60
    seconds before trying again to remove the taint.

```
student@lfs458-node-1a0a:~$ kubectl describe node | grep -i taint
Taints: node.kubernetes.io/not-ready:NoSchedule
Taints: <none>
```

#### 12 CHAPTER 3. INSTALLATION AND CONFIGURATION

```
student@lfs458-node-1a0a:~$ kubectl taint nodes \
--all node.kubernetes.io/not-ready-
```
```
node/lfs58-node-1a0a untainted
error: taint "node.kubernetes.io/not-ready:" not found
```
```
student@lfs458-node-1a0a:~$ sleep 60 ; kubectl describe node | grep -i taint
Taints: <none>
Taints: <none>
```
6. Determine if the DNS and Calico pods are ready for use. They should all show a status ofRunning. It may take a minute
    or two to transition fromPending.

```
student@lfs458-node-1a0a:~$ kubectl get pods --all-namespaces
NAMESPACE NAME READY STATUS RESTARTS AGE
kube-system calico-etcd-jlgwr 1/1 Running 0 6m
kube-system calico-kube-controllers-74b888b647-wlqf5 1/1 Running 0 6m
kube-system calico-node-tpvnr 2/2 Running 0 6m
kube-system coredns-78fcdf6894-nc5cn 1/1 Running 0 17m
kube-system coredns-78fcdf6894-xs96m 1/1 Running 0 17m
<output_omitted>
```
7. If you notice thecoredns-pods are stuck inContainerCreatingstatus you may have to delete them, causing new
    ones to be generated. Delete both pods and check to see they show aRunningstate. Your pod names will be different.

```
student@lfs458-node-1a0a:~$ kubectl get pods --all-namespaces
NAMESPACE NAME READY STATUS
RESTARTS AGE
kube-system calico-node-qkvzh 2/2 Running
0 59m
kube-system calico-node-vndn7 2/2 Running
0 12m
kube-system coredns-576cbf47c7-rn6v4 0/1 ContainerCreating
0 3s
kube-system coredns-576cbf47c7-vq5dz 0/1 ContainerCreating
0 94m
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ kubectl -n kube-system delete \
pod coredns-576cbf47c7-vq5dz coredns-576cbf47c7-rn6v
pod "coredns-576cbf47c7-vq5dz" deleted
pod "coredns-576cbf47c7-rn6v4" deleted
```
8. When it finished you should see a new tunnel,tunl0, and acaliinterface. It may take up to a minute to be created. As
    you create objects more interfaces will be created.

```
student@lfs458-node-1a0a:~$ ip a
<output_omitted>
4: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1440 qdisc noqueue state
UNKNOWN group default qlen 1000
link/ipip 0.0.0.0 brd 0.0.0.
inet 192.168.0.1/32 brd 192.168.0.1 scope global tunl
valid_lft forever preferred_lft forever
6: calib0b93ed4661@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu
1440 qdisc noqueue state UP group default
link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 1
inet6 fe80::ecee:eeff:feee:eeee/64 scope link
valid_lft forever preferred_lft forever
```
### Exercise 3.4: Deploy A Simple Application

We will test to see if we can deploy a simple application, in this case the **nginx** web server.


#### 3.1. LABS 13

1. Create a new deployment, which is an Kubernetes object while will deploy and monitor an application in a container.
    Verify it is running and the desired number of container matches the available.

```
student@lfs458-node-1a0a:~$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
```
```
student@lfs458-node-1a0a:~$ kubectl get deployments
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
nginx 1 1 1 1 6s
```
2. View the details of the deployment. Remember auto-completion will work for sub-commands and resources as well.

```
student@lfs458-node-1a0a:~$ kubectl describe deployment nginx
Name: nginx
Namespace: default
CreationTimestamp: Thu, 08 Nov 2018 19:23:00 +
Labels: app=nginx
Annotations: deployment.kubernetes.io/revision: 1
Selector: app=nginx
Replicas: 1 desired | 1 updated | 1 total | 1 ava....
StrategyType: RollingUpdate
MinReadySeconds: 0
RollingUpdateStrategy: 25% max unavailable, 25% max surge
<output_omitted>
```
3. View the basic steps the cluster took in order to pull and deploy the new application. You should see several lines of
    output with newer events at the top.

```
student@lfs458-node-1a0a:~$ kubectl get events
<output_omitted>
```
4. You can also view the output in **yaml** format, which could be used to create this deployment again or new deployments.
    Get the information but change the output to yaml. Note that halfway down there is status information of the current
    deployment.

```
student@lfs458-node-1a0a:~$ kubectl get deployment nginx -o yaml
apiVersion: extensions/v1beta
kind: Deployment
metadata:
annotations:
deployment.kubernetes.io/revision: "1"
creationTimestamp: 2017-09-27T18:21:25Z
<output_omitted>
```
5. Run the command again and redirect the output to a file. Then edit the file. Remove thecreationTimestamp,
    resourceVersion,selfLink, anduidlines. Also remove all the lines including and afterstatus:, which should
    be somewhere around line 40, if others have already been removed.

```
student@lfs458-node-1a0a:~$ kubectl get deployment nginx -o yaml > first.yaml
```
```
student@lfs458-node-1a0a:~$ vim first.yaml
```
```
<Remove the lines mentioned above>
```
6. Delete the existing deployment.

```
student@lfs458-node-1a0a:~$ kubectl delete deployment nginx
deployment.extensions "nginx" deleted
```
7. Create the deployment again this time using the file.

```
student@lfs458-node-1a0a:~$ kubectl create -f first.yaml
deployment.extension/nginx created
```

#### 14 CHAPTER 3. INSTALLATION AND CONFIGURATION

8. Look at the yaml output of this iteration and compare it against the first. Thetime stamp, resource version and
    uidwe had deleted are in the new file. These are generated for each resource we create, so we need to delete them
    from yaml files to avoid conflicts or false information. Thestatusshould not be hard-coded either.

```
student@lfs458-node-1a0a:~$ kubectl get deployment nginx -o yaml > second.yaml
```
```
student@lfs458-node-1a0a:~$ diff first.yaml second.yaml
<output_omitted>
```
9. Now that we have worked with the raw output we will explore two other ways of generating useful YAML or JSON. Use
    the--dry-runoption and verify no object was created. Only the priornginxdeployment should be found. The output
    lacks the unique information we removed before.

```
student@lfs458-node-1a0a:~$ kubectl create deployment two --image=nginx --dry-run -o yaml
apiVersion: apps/v1beta
kind: Deployment
metadata:
creationTimestamp: null
labels:
run: two
name: two
spec:
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ kubectl get deployment
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
nginx 1 1 1 1 7m
```
10. Existing objects can be viewed in a ready to use YAML output. Take a look at the existing **nginx** deployment. Note there
    is more detail to the **–export** option.

```
student@lfs458-node-1a0a:~$ kubectl get deployments nginx --export -o yaml
apiVersion: extensions/v1beta
kind: Deployment
metadata:
annotations:
deployment.kubernetes.io/revision: "1"
creationTimestamp: null
generation: 1
labels:
run: nginx
<output_omitted>
```
11. The output can also be viewed in JSON output.

```
student@lfs458-node-1a0a:~$ kubectl get deployment nginx --export -o json
{
"apiVersion": "extensions/v1beta1",
"kind": "Deployment",
"metadata": {
"annotations": {
"deployment.kubernetes.io/revision": "1"
},
<output_omitted>
```
12. The newly deployed **nginx** container is a light weight web server. We will need to create aserviceto view the default
    welcome page. Begin by looking at the help output. Note that there are several examples given, about halfway through
    the output.

```
student@lfs458-node-1a0a:~$ kubectl expose -h
<output_omitted>
```
13. Now try to gain access to the web server. As we have not declared a port to use you will receive an error.


#### 3.1. LABS 15

```
student@lfs458-node-1a0a:~$ kubectl expose deployment/nginx
error: couldn’t find port via --port flag or introspection
See ’kubectl expose -h’ for help and examples.
```
14. To change an existing configuration in a cluster can be done with subcommandsapply,editorpatchfor non-disruptive
    updates. Theapplycommand does a three-way diff of previous, current, and supplied input to determine modifications
    to make. Fields not mentioned are unaffected. Theeditfunction performs aget, opens an editor, then anapply. You
    can update API objects in place with JSON patch and merge patch or strategic merge patch functionality.
    If the configuration has resource fields which cannot be updated once initialized then a disruptive update could be done
    using thereplace --forceoption. This deletes first then re-creates a resource.
    Edit the file. Find the container name, somewhere around line 31 and add the port information as shown below.

```
student@lfs458-node-1a0a:~$ vim first.yaml
....
spec:
containers:
```
- image: nginx
    imagePullPolicy: Always
    name: nginx
    ports: # Add these
    - containerPort: 80 # three
       protocol: TCP # lines
    resources: {}
....
15. Due to how the object was created we will need to usereplaceto terminate and create a new deployment.

```
student@lfs458-node-1a0a:~$ kubectl replace -f first.yaml
deployment.extensions/nginx replaced
```
16. View the Pod and Deployment. Note theAGEshows the Pod was re-created.

```
student@lfs458-node-1a0a:~$ kubectl get deploy,pod
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
deployment.extensions/nginx 1 1 1 1 2m4s
```
```
NAME READY STATUS RESTARTS AGE
pod/nginx-7cbc4b4d9c-l8cgl 1/1 Running 0 8s
```
17. Try to expose the resource again. This time it should work.

```
student@lfs458-node-1a0a:~$ kubectl expose deployment/nginx
service/nginx exposed
```
18. Verify the service configuration. First look at the service information, then at the endpoint information. Note the Cluster
    IP is not the current endpoint. Take note of the current endpoint IP. In the example below it is10.244.1.99:80. We will
    use this information in a few steps.

```
student@lfs458-node-1a0a:~$ kubectl get svc nginx
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
nginx ClusterIP 10.100.61.122 <none> 80/TCP 3m
```
```
student@lfs458-node-1a0a:~$ kubectl get ep nginx
NAME ENDPOINTS AGE
nginx 10.244.1.99:80 4m
```
19. Determine which node the container is running on. Log into that node and use **tcpdump** to view traffic on thetunl0,
    as in tunnel zero, interface. The second node in this example. You may also see traffic on an interface which starts with
    caliand some string. Leave that command running while you run **curl** in the following step. You should see several
    messages go back and forth, including aHTTP: HTTP/1.1 200 OKand aackresponse to the same sequence.


#### 16 CHAPTER 3. INSTALLATION AND CONFIGURATION

```
student@lfs458-node-1a0a:~$ kubectl describe pod nginx-7cbc4b4d9c-d27xw \
| grep Node:
Node: lfs458-worker/10.128.0.5
```
```
student@lfs458-worker:~$ sudo tcpdump -i tunl0
tcpdump: verbose output suppressed, use -v or -vv for full protocol...
listening on tunl0, link-type EN10MB (Ethernet), capture size...
<output_omitted>
```
20. Test access to the Cluster IP, port 80. You should see the genericnginxinstalled and working page. The output should
    be the same when you look at theENDPOINTSIP address. If the **curl** command times out the pod may be running on
    the other node. Run the same command on that node and it should work.

```
student@lfs458-node-1a0a:~$ curl 10.100.61.122:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
<output_omitted>
```
21. Now scale up the deployment from one to three web servers.

```
student@lfs458-node-1a0a:~$ kubectl get deployment nginx
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
nginx 1 1 1 1 12m
```
```
student@lfs458-node-1a0a:~$ kubectl scale deployment nginx --replicas=3
deployment.extensions/nginx scaled
```
```
student@lfs458-node-1a0a:~$ kubectl get deployment nginx
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
nginx 3 3 3 3 12m
```
22. View the current endpoints. There now should be three. If theDESIREDabove said three, butAVAILABLEsaid two wait
    a few seconds and try again, it could be slow to fully deploy.

```
student@lfs458-node-1a0a:~$ kubectl get ep nginx
NAME ENDPOINTS AGE
nginx 10.244.0.66:80,10.244.1.100:80,10.244.1.99:80 10m
```
23. Find the oldest pod of the **nginx** deployment and delete it. TheTabkey can be helpful for the long names. Use theAGE
    field to determine which was running the longest. You will notice activity in the other terminal where **tcpdump** is running,
    when you delete the pod.

```
student@lfs458-node-1a0a:~$ kubectl get po -o wide
NAME READY STATUS RESTARTS AGE IP
nginx-1423793266-7f1qw 1/1 Running 0 14m 10.244.0.66
nginx-1423793266-8w2nk 1/1 Running 0 1m 10.244.1.100
nginx-1423793266-fbt4b 1/1 Running 0 1m 10.244.1.101
```
```
student@lfs458-node-1a0a:~$ kubectl delete po nginx-1423793266-7f1qw
pod "nginx-1423793266-7f1qw" deleted
```
24. Wait a minute or two then view the pods again. One should be newer than the others. In the following example two
    minutes instead of four. If your **tcpdump** was using thevethinterface of that container it will error out.

```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
nginx-1423793266-13p69 1/1 Running 0 2m
nginx-1423793266-8w2nk 1/1 Running 0 4m
nginx-1423793266-fbt4b 1/1 Running 0 4m
```

#### 3.1. LABS 17

25. View the endpoints again. The original endpoint IP is no longer in use. You can delete any of the pods and theservice
    will forward traffic to the existing backend pods.

```
student@lfs458-node-1a0a:~$ kubectl get ep nginx
NAME ENDPOINTS AGE
nginx 10.244.0.66:80,10.244.1.100:80,10.244.1.101:80 15m
```
26. Test access to the web server again, using theClusterIPaddress, then any of the endpoint IP addresses. Even though
    the endpoints have changed you still have access to the web server. This access is only from within the cluster.

```
student@lfs458-node-1a0a:~$ curl 10.100.61.122:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
body {
<output_omitted>
```
### Exercise 3.5: Access from Outside the Cluster

You can access a Service from outside the cluster using a DNS add-on or **vi** environment variables. We will use environment
variables to gain access to a Pod.

1. Begin by getting a list of the pods.

```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
nginx-1423793266-13p69 1/1 Running 0 8m
nginx-1423793266-8w2nk 1/1 Running 0 10m
nginx-1423793266-fbt4b 1/1 Running 0 10m
```
2. Choose one of the pods and use the exec command to run **printenv** inside the pod. The following example uses the
    first pod listed above.

```
student@lfs458-node-1a0a:~$ kubectl exec nginx-1423793266-13p69 \
-- printenv |grep KUBERNETES
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
NGINX_SERVICE_HOST=10.100.61.122
NGINX_SERVICE_PORT=80
<output_omitted>
```
3. Find and then delete the existing service for **nginx**.

```
student@lfs458-node-1a0a:~$ kubectl get svc
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 10.96.0.1 <none> 443/TCP 2d
nginx ClusterIP 10.100.61.122 <none> 80/TCP 40m
```
4. Delete the service.

```
student@lfs458-node-1a0a:~$ kubectl delete svc nginx
service "nginx" deleted
```
5. Create the service again, but this time pass theLoadBalancertype. Check to see the status and note the external ports
    mentioned. The output will show theExternal-IPaspending. Unless a provider responds with a load balancer it will
    continue to show as pending.


#### 18 CHAPTER 3. INSTALLATION AND CONFIGURATION

```
student@lfs458-node-1a0a:~$ kubectl expose deployment nginx --type=LoadBalancer
service/nginx exposed
```
```
student@lfs458-node-1a0a:~$ kubectl get svc
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 10.96.0.1 <none> 443/TCP 2d
nginx LoadBalancer 10.104.249.102 <pending> 80:32753/TCP 2s
```
6. Open a browser on your local system, not the GCE node, and use the public IP of your node and port 32753 , shown
    in the output above. If running the labs on a remote system like **AWS** or **GCE** the CLUSTER-IPs are internal. Use the
    public IP you used with SSH to gain access.

```
Figure 3.1: External Access via Browser
```
7. Scale the deployment to zero replicas. Then test the web page again. It should fail.

```
student@lfs458-node-1a0a:~$ kubectl scale deployment nginx --replicas=0
deployment.extensions/nginx scaled
```
```
student@lfs458-node-1a0a:~$ kubectl get po
No resources found.
```
8. Scale the deployment up to two replicas. The web page should work again.

```
student@lfs458-node-1a0a:~$ kubectl scale deployment nginx --replicas=2
deployment.extensions/nginx scaled
```
```
student@lfs458-node-1a0a:~$ kubectl get po
.NAME READY STATUS RESTARTS AGE
nginx-1423793266-7x181 1/1 Running 0 1m
nginx-1423793266-s6vcz 1/1 Running 0 1m
```
9. Delete the deployment to recover system resources. Note that deleting adeploymentdoes not delete the endpoints or
    services.

```
student@lfs458-node-1a0a:~$ kubectl delete deployments nginx
deployment.extensions "nginx" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl delete ep nginx
endpoints "nginx" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl delete svc nginx
service "nginx" deleted
```

**Chapter 4**

**Kubernetes Architecture**

### 4.1 Labs

### Exercise 4.1: Working with CPU and Memory Constraints

### Overview

We will continue working with our cluster, which we built in the previous lab. We will work withresource limits, more with
namespacesand then a complexdeploymentwhich you can explore to further understand the architecture and relationships.

Use **SSH** or **PuTTY** to connect to the nodes you installed in the previous exercise. We will deploy an application called **stress**
inside a container, and then useresource limitsto constrain the resources the application has access to use.

1. Use a container calledstress, which we will namehog, to generate load. Verify you have a container running.

```
student@lfs458-node-1a0a:~$ kubectl create deployment hog --image vish/stress
deployment.apps/hog created
```
```
student@lfs458-node-1a0a:~$ kubectl get deployments
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
hog 1 1 1 1 12s
```
2. Use thedescribeargument to view details, then view the output inYAMLformat. Note there are no settings limiting
    resource usage. Instead, there are empty curly brackets.

```
student@lfs458-node-1a0a:~$ kubectl describe deployment hog
Name: hog
Namespace: default
CreationTimestamp: Fri, 09 Nov 2018 19:55:45 +0000
Labels: app=hog
Annotations: deployment.kubernetes.io/revision: 1
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ kubectl get deployment hog -o yaml
apiVersion: extensions/v1beta1
```
#### 19


#### 20 CHAPTER 4. KUBERNETES ARCHITECTURE

```
kind: Deployment
Metadata:
```
```
<output_omitted>
```
```
template:
metadata:
creationTimestamp: null
labels:
app: hog
spec:
containers:
```
- image: vish/stress
    imagePullPolicy: Always
    name: stress
    resources: {}
    terminationMessagePath: /dev/termination-log
<output_omitted>
3. We will use theYAMLoutput to create our own configuration file. The--exportoption can be useful to not include
unique parameters.

```
student@lfs458-node-1a0a:~$ kubectl get deployment hog \
--export -o yaml > hog.yaml
```
4. If you did not use the--exportoption we will need to remove thestatusoutput,creationTimestampand other
    settings, as we don’t want to set unique generated parameters. We will also add in memory limits found below.

```
student@lfs458-node-1a0a:~$ vim hog.yaml
.
imagePullPolicy: Always
name: hog
resources: # Edit to remove {}
limits: # Add these 4 lines
memory: "4Gi"
requests:
memory: "2500Mi"
terminationMessagePath: /dev/termination-log
terminationMessagePolicy: File
....
```
5. Replace the deployment using the newly edited file.

```
student@lfs458-node-1a0a:~$ kubectl replace -f hog.yaml
deployment.extensions/hog replaced
```
6. Verify the change has been made. The deployment should now show resource limits.

```
student@lfs458-node-1a0a:~$ kubectl get deployment hog -o yaml |less
....
resources:
limits:
memory: 4Gi
requests:
memory: 2500Mi
terminationMessagePath: /dev/termination-log
....
```
7. View thestdioof the hog container. Note how how much memory has been allocated.


#### 4.1. LABS 21

```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
hog-64cbfcc7cf-lwq66 1/1 Running 0 2m
```
```
student@lfs458-node-1a0a:~$ kubectl logs hog-64cbfcc7cf-lwq66
I1102 16:16:42.638972 1 main.go:26] Allocating "0" memory, in
"4Ki" chunks, with a 1ms sleep between allocations
I1102 16:16:42.639064 1 main.go:29] Allocated "0" memory
```
8. Open a second and third terminal to access both master and second nodes. Run **top** to view resource usage. You
    should not see unusual resource usage at this point. The **dockerd** and **top** processes should be using about the same
    amount of resources. The **stress** command should not be using enough resources to show up.
9. Edit thehogconfiguration file and add arguments for **stress** to consume CPU and memory.

```
student@lfs458-node-1a0a:~$ vim hog.yaml
resources:
limits:
cpu: "1"
memory: "4Gi"
requests:
cpu: "0.5"
memory: "500Mi"
args:
```
- -cpus
- "2"
- -mem-total
- "950Mi"
- -mem-alloc-size
- "100Mi"
- -mem-alloc-sleep
- "1s"
10. Delete and recreate the deployment. You should see CPU usage almost immediately and memory allocation happen in
100M chunks allocated to the **stress** program. Check both nodes as the container could deployed to either. The next
step will help if you have errors.

```
student@lfs458-node-1a0a:~$ kubectl delete deployment hog
deployment.extensions/hog deleted
```
```
student@lfs458-node-1a0a:~$ kubectl apply -f hog.yaml
deployment.extensions/hog created
```
11. Should the resources not show as used, there may have been an issue inside of the container. Kubernetes shows it
    as running, but the actual workload has failed. Or the container may have failed; for example if you were missing a
    parameter the container may panic and show the following output.

```
student@lfs458-node-1a0a:~$ kubectl get pod
NAME READY STATUS RESTARTS AGE
hog-1985182137-5bz2w 0/1 Error 1 5s
```
```
student@lfs458-node-1a0a:~$ kubectl logs hog-1985182137-5bz2w
panic: cannot parse ’150mi’: unable to parse quantity’s suffix
```
```
goroutine 1 [running]:
panic(0x5ff9a0, 0xc820014cb0)
/usr/local/go/src/runtime/panic.go:481 +0x3e6
k8s.io/kubernetes/pkg/api/resource.MustParse(0x7ffe460c0e69, 0x5, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0)
/usr/local/google/home/vishnuk/go/src/k8s.io/kubernetes/pkg/api/resource/quantity.go:134 +0x287
main.main()
/usr/local/google/home/vishnuk/go/src/github.com/vishh/stress/main.go:24 +0x43
```

#### 22 CHAPTER 4. KUBERNETES ARCHITECTURE

12. Here is an example of an improper parameter. The container is running, but not allocating memory. It should show the
    usage requested from the YAML file.

```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
hog-1603763060-x3vnn 1/1 Running 0 8s
```
```
student@lfs458-node-1a0a:~$ kubectl logs hog-1603763060-x3vnn
I0927 21:09:23.514921 1 main.go:26] Allocating "0" memory, in "4Ki" chunks, with a 1ms sleep \
between allocations
I0927 21:09:23.514984 1 main.go:39] Spawning a thread to consume CPU
I0927 21:09:23.514991 1 main.go:39] Spawning a thread to consume CPU
I0927 21:09:23.514997 1 main.go:29] Allocated "0" memory
```
### Exercise 4.2: Resource Limits for a Namespace

The previous steps set limits for that particulardeployment. You can also set limits on an entirenamespace. We will create
a newnamespaceand configure the **hog** deployment to run within. When set **hog** should not be able to use the previous
amount of resources.

1. Begin by creating a new namespace calledlow-usage-limitand verify it exists.

```
student@lfs458-node-1a0a:~$ kubectl create namespace low-usage-limit
namespace/low-usage-limit created
```
```
student@lfs458-node-1a0a:~$ kubectl get namespace
NAME STATUS AGE
default Active 1h
kube-public Active 1h
kube-system Active 1h
low-usage-limit Active 42s
```
2. Create a YAML file which limits CPU and memory usage. The kind to use isLimitRange.

```
student@lfs458-node-1a0a:~$ vim low-resource-range.yaml
```
```
apiVersion: v1
kind: LimitRange
metadata:
name: low-resource-range
spec:
limits:
```
- default:
    cpu: 1
    memory: 500Mi
defaultRequest:
cpu: 0.5
memory: 100Mi
type: Container
3. Create the LimitRange object and assign it to the newly created namespace low-usage-limit

```
student@lfs458-node-1a0a:~$ kubectl create -f low-resource-range.yaml \
--namespace=low-usage-limit
limitrange/low-resource-range created
```
4. Verify it works. Remember that every command needs a namespace and context to work. Defaults are used if not
    provided.


#### 4.1. LABS 23

```
student@lfs458-node-1a0a:~$ kubectl get LimitRange
No resources found.
```
```
student@lfs458-node-1a0a:~$ kubectl get LimitRange --all-namespaces
NAMESPACE NAME CREATED AT
low-usage-limit low-resource-range 2018-07-08T06:28:33Z
```
5. Create a new deployment in the namespace.

```
student@lfs458-node-1a0a:~$ kubectl create deployment limited-hog \
--image vish/stress -n low-usage-limit
deployment.apps/limited-hog created
```
6. List the current deployments. Notehogcontinues to run in the default namespace. If you chose to use the **Calico**
    network policy you may see a couple more than what is listed below.

```
student@lfs458-node-1a0a:~$ kubectl get deployments --all-namespaces
NAMESPACE NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
default hog 1 1 1 1 25m
kube-system kube-dns 1 1 1 1 2d
low-usage-limit limited-hog 1 1 1 1 1m
```
7. View all pods within the namespace. Remember you can use the **tab** key to complete the namespace. You may want to
    type the namespace first so that tab-completion is appropriate to that namespace instead of thedefaultnamespace.

```
student@lfs458-node-1a0a:~$ kubectl -n low-usage-limit get pods
NAME READY STATUS RESTARTS AGE
limited-hog-2556092078-wnpnv 1/1 Running 0 3m
```
8. Look at the details of the pod. You will note it has the settings inherited from the entire namespace. The use of shell
    completion should work if you declare the namespace first.

```
student@lfs459-node-1a0a:~$ kubectl -n low-usage-limit get pod limited-hog-2556092078-wnpnv -o yaml
<output_omitted>
spec:
containers:
```
- image: vish/stress
    imagePullPolicy: Always
    name: stress
    resources:
       limits:
          cpu: "1"
          memory: 500Mi
       requests:
          cpu: 500m
          memory: 100Mi
    terminationMessagePath: /dev/termination-log
<output_omitted>
9. Copy and edit the config file for the originalhogfile. Add the namespace: line so that a new deployment would be in the
low-usage-limitnamespace.

```
student@lfs458-node-1a0a:~$ cp hog.yaml hog2.yaml
```
```
student@lfs458-node-1a0a:~$ vim hog2.yaml
....
labels:
app: hog
name: hog
namespace: low-usage-limit #<<--- Add this line
selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/hog
spec:
....
```

#### 24 CHAPTER 4. KUBERNETES ARCHITECTURE

10. Open up extra terminal sessions so you can have **top** running in each. When the new deployment is created it will
    probably be scheduled on the node not yet under any stress.
    Create the deployment.

```
student@lfs458-node-1a0a:~$ kubectl create -f hog2.yaml
deployment.extensions/hog created
```
11. View the deployments. Note there are two with the same name, but in different namespaces. You may also find the
    calico-typhadeployment has no pods, nor has any requested. Our small cluster does not need to add **Calico** pods
    via this autoscaler.

```
student@lfs458-node-1a0a:~$ kubectl get deployments --all-namespaces
NAMESPACE NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
default hog 1 1 1 1 45m
kube-system calico-typha 0 0 0 0 8h
kube-system coredns 2 2 2 2 8h
low-usage-limit hog 1 1 1 1 13s
low-usage-limit limited-hog 1 1 1 1 5m
```
12. Look at the **top** output running in other terminals. You should find that bothhogdeployments are using about the
    same amount of resources, once the memory is fully allocated. Per-deployment settings override the global namespace
    settings. You should see something like the following lines one from each node, which indicates use of one processor
    and about 12 percent of your memory, were you on a system with 8G total.

```
25128 root 20 0 958532 954672 3180 R 100.0 11.7 0:52.27 stress
```
```
24875 root 20 0 958532 954800 3180 R 100.3 11.7 41:04.97 stress
```
13. Delete thehogdeployments to recover system resources.

```
student@lfs458-node-1a0a:~$ kubectl -n low-usage-limit delete deployment hog
deployment.extensions "hog" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl delete deployment hog
deployment.extensions "hog" deleted
```
### Exercise 4.3: More Complex Deployment

We will now deploy a more complex demo application to test the cluster. When completed it will be a sock shopping site.
The short URL is shown below for:https://raw.githubusercontent.com/microservices-demo/microservices-demo/
master/deploy/kubernetes/complete-demo.yaml

1. Begin by downloading the pre-made YAML file from github.

```
student@lfs458-node-1a0a:~$ wget https://tinyurl.com/y8bn2awp -O complete-demo.yaml
Resolving tinyurl.com (tinyurl.com)... 104.20.218.42, 104.20.219.42,
Connecting to tinyurl.com (tinyurl.com)|104.20.218.42|:443... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://raw.githubusercontent.com/microservices-demo/microservices-...
--2017-11-02 16:54:27-- https://raw.githubusercontent.com/microservices-dem...
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.5...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101....
HTTP request sent, awaiting response... 200 OK
<output_omitted>
```
2. Find the expected namespaces inside the file. It should besock-shop. Also note the various settings. This file will
    deploy several containers which work together, providing a shopping website. As we work with other parameters you
    could revisit this file to see potential settings.


#### 4.1. LABS 25

```
student@lfs458-node-1a0a:~$ less complete-demo.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
name: carts-db
labels:
name: carts-db
namespace: sock-shop
spec:
replicas: 1
<output_omitted>
```
3. Create the namespace and verify it was made.

```
student@lfs458-node-1a0a:~$ kubectl create namespace sock-shop
namespace/sock-shop created
```
```
student@lfs458-node-1a0a:~$ kubectl get namespace
NAME STATUS AGE
default Active 35m
kube-public Active 35m
kube-system Active 35m
low-usage-limit Active 25m
sock-shop Active 5s
```
4. View the images the new application will deploy.

```
student@lfs458-node-1a0a:~$ grep image complete-demo.yaml
image: mongo
image: weaveworksdemos/carts:0.4.8
image: weaveworksdemos/catalogue-db:0.3.0
image: weaveworksdemos/catalogue:0.3.5
image: weaveworksdemos/front-end:0.3.12
image: mongo
<output_omitted>
```
5. Create the new shopping website using the YAML file. Use the namespace you recently created. Note that the deploy-
    ments match the images we saw in the file.

```
student@lfs458-node-1a0a:~$ kubectl apply -n sock-shop -f complete-demo.yaml
deployment "carts-db" created
service "carts-db" created
deployment "carts" created
service "carts" created
<output_omitted>
```
6. Using the proper namespace will be important. This can be set on a per-command basis or as a shell parameter. Note
    the first command shows no pods. We must remember to pass the proper namespace. Some containers may not have
    fully downloaded or deployed by the time you run the command.

```
student@lfs458-node-1a0a:~$ kubectl get pods
No resources found.
```
```
student@lfs458-node-1a0a:~$ kubectl -n sock-shop get pods
NAME READY STATUS RESTARTS AGE
carts-511261774-c4jwv 1/1 Running 0 71s
carts-db-549516398-tw9zs 1/1 Running 0 71s
catalogue-4293036822-sp5kt 1/1 Running 0 71s
catalogue-db-1846494424-qzhvk 1/1 Running 0 71s
front-end-2337481689-6s65c 1/1 Running 0 71s
orders-208161811-1gc6k 1/1 Running 0 71s
orders-db-2069777334-4sp01 1/1 Running 0 71s
payment-3050936124-2cn2l 1/1 Running 0 71s
```

#### 26 CHAPTER 4. KUBERNETES ARCHITECTURE

```
queue-master-2067646375-vzq77 1/1 Running 0 71s
rabbitmq-241640118-vk3m9 0/1 ContainerCreating 0 71s
shipping-3132821717-lm7kn 0/1 ContainerCreating 0 71s
user-1574605338-24xrb 0/1 ContainerCreating 0 71s
user-db-2947298815-lx9kp 1/1 Running 0 71s
```
7. Verify the shopping cart is exposing a web page. Use the public IP address of your AWS node (not the one derived from
    the prompt) to view the page. Note the external IP is not yet configured. Find theNodePortservice. First try port 80
    then try port 30001 as shown under thePORTScolumn.

```
student@lfs458-node-1a0a:~$ kubectl get svc -n sock-shop
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
carts ClusterIP 10.100.154.148 <none> 80/TCP 95s
carts-db ClusterIP 10.111.120.73 <none> 27017/TCP 95s
catalogue ClusterIP 10.100.8.203 <none> 80/TCP 95s
catalogue-db ClusterIP 10.111.94.74 <none> 3306/TCP 95s
front-end NodePort 10.98.2.137 <none> 80:30001/TCP 95s
orders ClusterIP 10.110.7.215 <none> 80/TCP 95s
orders-db ClusterIP 10.106.19.121 <none> 27017/TCP 95s
payment ClusterIP 10.111.28.218 <none> 80/TCP 95s
queue-master ClusterIP 10.102.181.253 <none> 80/TCP 95s
rabbitmq ClusterIP 10.107.134.121 <none> 5672/TCP 95s
shipping ClusterIP 10.99.99.127 <none> 80/TCP 95s
user ClusterIP 10.105.126.10 <none> 80/TCP 95s
user-db ClusterIP 10.99.123.228 <none> 27017/TCP 95s
```
8. Check to see which node is running the containers. Note that the webserver is answering on a node which is not hosting
    the all the containers. First we check the master, then the second node. The containers should have to do with **kube**
    **proxy** services and **calico**. The following is the **sudo docker ps** on both nodes.

```
student@lfs458-node-1a0a:~$ sudo docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
d6b7353e5dc5 weaveworksdemos/user@sha256:2ffccc332963c89e035fea52201012208bf62df43a55fe461ad6598a5c757ab7 "/user -port=80" 2 minutes ago Up 2 minutes k8s_user_user-7848fb86db-5zmkj_sock-shop_584d7db5-947b-11e8-8cfb-42010a800002_0
6c18f030f15b weaveworksdemos/shipping@sha256:983305c948fded487f4a4acdeab5f898e89d577b4bc1ca3de7750076469ccad4 "/usr/local/bin/ja..." 2 minutes ago Up 2 minutes k8s_shipping_shipping-64f8c7558c-9kgm2_sock-shop_580a50f9-947b-11e8-8cfb-42010a800002_0
baaa8d67ebef weaveworksdemos/queue-master@sha256:6292d3095f4c7aeed8d863527f8ef6d7a75d3128f20fc61e57f398c100142712 "/usr/local/bin/ja..." 2 minutes ago Up 2 minutes k8s_queue-master_queue-master-787b68b7fd-2tld8_sock-shop_57dca0ab-947b-11e8-8cfb-42010a800002_0
<output_omitted>
```
```
student@lfs458-worker:~$ sudo docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
9452559caa0d weaveworksdemos/payment@sha256:5ab1c9877480a018d4dda10d6dfa382776e6bca9fc1c60bacbb80903fde8cfe0 "/app -port=80" 2 minutes ago Up 2 minutes k8s_payment_payment-5df6dc6bcc-k2hbl_sock-shop_57c79b30-947b-11e8-8cfb-42010a800002_0
993017c7b476 weaveworksdemos/user-db@sha256:b43f0f8a76e0c908805fcec74d1ad7f4af4d93c4612632bd6dc20a87508e0b68 "/entrypoint.sh mo..." 2 minutes ago Up 2 minutes k8s_user-db_user-db-586b8566b4-j7f24_sock-shop_58418841-947b-11e8-8cfb-42010a800002_0
1356b0548ee8 weaveworksdemos/orders@sha256:b622e40e83433baf6374f15e076b53893f79958640fc6667dff597622eff03b9 "/usr/local/bin/ja..." 2 minutes ago Up 2 minutes k8s_orders_orders-5c4f477565-gzh7x_sock-shop_57bf7576-947b-11e8-8cfb-42010a800002_0
<output_omitted>
```
9. Now we will shut down the shopping application. This can be done a few different ways. Begin by getting a listing of
    resources in all namespaces. There should be about 14 deployments.

```
student@lfs458-node-1a0a:~$ kubectl get deployment --all-namespaces
NAMESPACE NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
kube-system calico-typha 0 0 0 0 33m
kube-system coredns 2 2 2 2 33m
low-usage-limit limited-hog 1 1 1 1 33m
sock-shop carts 1 1 1 1 6m44s
sock-shop carts-db 1 1 1 1 6m44s
sock-shop catalogue 1 1 1 1 6m44s
<output_omitted>
```
10. Use the terminal on the second node to get a count of the current docker containers. It should be something like 30,
    plus a line for status counted by **wc**. The main system should have something like 26 running, plus a line of status.

```
student@lfs458-node-1a0a:~$ sudo docker ps | wc -l
26
```

#### 4.1. LABS 27

```
student@lfs458-worker:~$ sudo docker ps | wc -l
30
```
11. In order to complete maintainence we may need to move containers from a node and prevent new ones from deploying.
    One way to do this is to **drain** , orcordon, the node. Currently this will not affectDaemonSets, an object we will discuss
    in greater detail in the future. Begin by getting a list of nodes. Your node names will be different.

```
student@lfs458-node-1a0a:~$ kubectl get nodes
NAME STATUS ROLES AGE VERSION
lfs458-worker Ready <none> 3h v1.12.1
lfs458-node-1a0a Ready master 3h v1.12.1
```
12. Modifying your second, worker node, update the node to **drain** the pods. Some resources may not drain, expect an
    error which we will work with next. Note the error includesaborting commandwhich indicates the drain did not take
    place. Were you to check it would have the same number of containers running, but will show a new taint preventing the
    scheduler from assigning new pods.

```
student@lfs458-node-1a0a:~$ kubectl drain lfs458-worker
node/lfs458-worker cordoned
error: unable to drain node "lfs458-worker", aborting command...
```
```
There are pending nodes to be drained:
lfs458-worker
error: DaemonSet-managed pods (use --ignore-daemonsets to ignore):
calico-node-vndn7, kube-proxy-rjjls
```
```
student@lfs458-node-1a0a:~$ kubectl describe node |grep -i taint
Taints: <none>
Taints: node.kubernetes.io/unschedulable:NoSchedule
```
13. As the error output suggests we can use the **–ignore-daemonsets** options to ignore containers which are not intended
    to move. We will find a new error when we use this command, near the end of the output. The node will continue to
    have the same number of pods and containers running.

```
student@lfs458-node-1a0a:~$ kubectl drain lfs458-worker --ignore-daemonsets
node/worker cordoned
error: unable to drain node "lfs458-worker", aborting command...
```
```
There are pending nodes to be drained:
lfs458-worker
error: pods with local storage (use --delete-local-data to override):
carts-55f7f5c679-ffkq2, carts-db-5c55874946-w728d, orders-7b69bf5686-vtkcn
```
14. Run the command again. This time the output should both indicate the node has already been cordoned, then show the
    eviction of several pods. Not all pods will be gone asdaemonsetswill remain. Note the command is shown on two lines.
    You can omit the backslash and type the command on a single line.

```
student@lfs458-node-1a0a:~$ kubectl drain lfs458-worker \
--ignore-daemonsets --delete-local-data
```
```
node/lfs458-worker already cordoned
WARNING: Ignoring DaemonSet-managed pods: calico-node-vndn7, kube-proxy-rjjls; Deleting pods with local storage: carts-55f7f5c679-ppv7p, carts-db-5c55874946-h42v2, orders-7b69bf5686-t82lz, orders-db-7bc46bdb98-x5zrl
pod/carts-db-5c55874946-h42v2 evicted
pod/orders-db-7bc46bdb98-x5zrl evicted
pod/catalogue-db-66ff5bbbf5-2wmx4 evicted
pod/catalogue-5764fdf6d-8gk96 evicted
pod/orders-7b69bf5686-t82lz evicted
pod/front-end-f99dbcb9c-92q4p evicted
pod/carts-55f7f5c679-ppv7p evicted
```
15. Were you to look on your second, worker node, you would see there should be fewer pods and containers than before.
    These pods can only be evicted via a special taint which we will discuss in the scheduling chapter.


#### 28 CHAPTER 4. KUBERNETES ARCHITECTURE

```
student@lfs458-worker:~$ sudo docker ps | wc -l
6
```
16. Update the node taint such that the scheduler will use the node again. Verify that no nodes have moved over to the
    worker node as the scheduler only checks when a pod is deployed.

```
student@lfs458-node-1a0a:~$ kubectl uncordon lfs458-worker
node/lfs458-worker uncordoned
```
```
student@lfs458-node-1a0a:~$ kubectl describe node |grep -i taint
Taints: <none>
Taints: <none>
```
```
student@lfs458-worker:~$ sudo docker ps | wc -l
6
```
17. As we clean up our sock shop let us see some differences between pods and deployments. Start with a list of the pods
    that are running in thesock-shopnamespace.

```
student@lfs458-node-1a0a:~$ kubectl -n sock-shop get pod
NAME READY STATUS RESTARTS AGE
carts-db-549516398-tw9zs 1/1 Running 0 6h
catalogue-4293036822-sp5kt 1/1 Running 0 6h
<output_omitted>
```
18. Delete a few resources using the pod name.

```
student@lfs458-node-1a0a:~$ kubectl -n sock-shop delete pod \
catalogue-4293036822-sp5kt catalogue-db-1846494424-qzhvk \
front-end-2337481689-6s65c orders-208161811-1gc6k \
orders-db-2069777334-4sp01
pod "catalogue-4293036822-sp5kt" deleted
pod "catalogue-db-1846494424-qzhvk" deleted
<output_omitted>
```
19. Check the status of the pods. There should be some pods running for only a few seconds. These will have the same
    name-stub as the Pods you recently deleted. TheDeploymentcontroller noticed expected number of Pods was not
    proper, so created new Pods until the current state matches the Pod manifest.

```
student@lfs458-node-1a0a:~$ kubectl -n sock-shop get pod
NAME READY STATUS RESTARTS AGE
catalogue-4293036822-mtz8m 1/1 Running 0 22s
catalogue-db-1846494424-16n2p 1/1 Running 0 22s
front-end-2337481689-6s65c 1/1 Terminating 0 6h
front-end-2337481689-80gwt 1/1 Running 0 22s
```
20. Delete some of the resources via deployments.

```
student@lfs458-node-1a0a:~$ kubectl -n sock-shop delete deployment \
catalogue catalogue-db front-end orders
deployment "catalogue" deleted
deployment "catalogue-db" deleted
```
21. Check and both the pods and deployments you removed have not been recreated.

```
student@lfs458-node-1a0a:~$ kubectl -n sock-shop get pods |grep catalogue
```
```
student@lfs458-node-1a0a:~$ kubectl -n sock-shop get deployment
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
carts 1 1 1 1 71m
carts-db 1 1 1 1 71m
orders-db 1 1 1 1 71m
payment 1 1 1 1 71m
```

#### 4.1. LABS 29

```
queue-master 1 1 1 1 71m
rabbitmq 1 1 1 1 71m
shipping 1 1 1 1 71m
user 1 1 1 1 71m
user-db 1 1 1 1 71m
```
22. Delete the rest of the deployments. When no resources are found, examine the output of thedocker pscommand.
    None of thesock-shopcontainers should be found. Use the same file we created with to delete all of the objects made.
    You will get some errors because we deleted a few deployments by hand.

```
student@lfs458-node-1a0a:~$ kubectl delete -f complete-demo.yaml
<output_omitted>
```

#### 30 CHAPTER 4. KUBERNETES ARCHITECTURE


**Chapter 5**

**APIs and Access**

### 5.1 Labs

### Exercise 5.1: Configuring TLS Access

### Overview

Using the Kubernetes API, **kubectl** makes API calls for you. With the appropriate TLS keys you could run **curl** as well use a
**golang** client. Calls to thekube-apiserverget or set a PodSpec, or desired state. If the request represents a new state the
**Kubernetes Control Plane** will update the cluster until the current state matches the specified state. Some end states may
require multiple requests. For example, to delete aReplicaSet, you would first set the number of replicas to zero, then delete
theReplicaSet.

An API request must pass information as JSON. **kubectl** converts.yamlto JSON when making an API request on your
behalf. The API request has many settings, but must includeapiVersion,kindandmetadata, andspecsettings to declare
what kind of container to deploy. Thespecfields depend on the object being created.

We will begin by configuring remote access to thekube-apiserverthen explore more of the API.

### Configuring TLS Access

1. Begin by reviewing the **kubectl** configuration file. We will use the three certificates and the API server address.

```
student@lfs458-node-1a0a:~$ less ~/.kube/config
<output_omitted>
```
2. We will set the certificates as variables. You may want to double-check each parameter as you set it. Begin with setting
    theclient-certificate-datakey.

```
student@lfs458-node-1a0a:~$ export client=$(grep client-cert ~/.kube/config |cut -d" " -f 6)
```
```
student@lfs458-node-1a0a:~$ echo $client
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJ
```
#### 31


#### 32 CHAPTER 5. APIS AND ACCESS

```
BZ0lJRy9wbC9rWEpNdmd3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0
ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB4TnpFeU1UTXhOelEyTXpKY
UZ3MHhPREV5TVRNeE56UTJNelJhTURReApGekFWQmdOVkJBb1REbk41YzNS
<output_omitted>
```
3. Almost the same command, but this time collect theclient-key-dataas thekeyvariable.

```
student@lfs458-node-1a0a:~$ export key=$(grep client-key-data ~/.kube/config |cut -d " " -f 6)
```
```
student@lfs458-node-1a0a:~$ echo $key
<output_omitted>
```
4. Finally set theauthvariable with thecertificate-authority-datakey.

```
student@lfs458-node-1a0a:~$ export auth=$(grep certificate-authority-data ~/.kube/config |cut -d " " -f 6)
```
```
student@lfs458-node-1a0a:~$ echo $auth
<output_omitted>
```
5. Now encode the keys for use with **curl**.

```
student@lfs458-node-1a0a:~$ echo $client | base64 -d - > ./client.pem
```
```
student@lfs458-node-1a0a:~$ echo $key | base64 -d - > ./client-key.pem
```
```
student@lfs458-node-1a0a:~$ echo $auth | base64 -d - > ./ca.pem
```
6. Pull the API server URL from the config file.

```
student@lfs458-node-1a0a:~$ kubectl config view |grep server
server: https://10.128.0.3:6443
```
7. Use **curl** command and the encoded keys to connect to the API server.

```
student@lfs458-node-1a0a:~$ curl --cert ./client.pem \
--key ./client-key.pem \
--cacert ./ca.pem \
https://10.128.0.3:6443/api/v1/pods
```
```
{
"kind": "PodList",
"apiVersion": "v1",
"metadata": {
"selfLink": "/api/v1/pods",
"resourceVersion": "239414"
},
<output_omitted>
```
8. If the previous command was successful, create a JSON file to create a new pod. Remember to look for this file in the
    tarball output, it can save you some typing.

```
student@lfs458-node-1a0a:~$ vim curlpod.json
{
"kind": "Pod",
"apiVersion": "v1",
"metadata":{
"name": "curlpod",
"namespace": "default",
"labels": {
"name": "examplepod"
}
},
"spec": {
"containers": [{
```

#### 5.1. LABS 33

```
"name": "nginx",
"image": "nginx",
"ports": [{"containerPort": 80}]
}]
}
}
```
9. The previous **curl** command can be used to build aXPOSTAPI call. There will be a lot of output, including the scheduler
    and taints involved. Read through the output. In the last few lines the phase will probably showPending, as it’s near the
    beginning of the creation process.

```
student@lfs458-node-1a0a:~$ curl --cert ./client.pem \
--key ./client-key.pem --cacert ./ca.pem \
https://10.128.0.3:6443/api/v1/namespaces/default/pods \
-XPOST -H’Content-Type: application/json’ \
-d@curlpod.json
```
```
{
"kind": "Pod",
"apiVersion": "v1",
"metadata": {
"name": "curlpod",
<output_omitted>
```
10. Verify the new pod exists and shows aRunningstatus.

```
student@lfs458-node-1a0a:~$ kubectl get pods
NAME READY STATUS RESTARTS AGE
curlpod 1/1 Running 0 45s
```
### Exercise 5.2: Explore API Calls

1. One way to view what a command does on your behalf is to use **strace**. In this case, we will look for the current
    endpoints, or targets of our API calls.

```
student@lfs458-node-1a0a:~$ kubectl get endpoints
NAME ENDPOINTS AGE
kubernetes 10.128.0.3:6443 3h
```
2. Run this command again, preceded by **strace**. You will get a lot of output. Near the end you will note several **openat**
    functions to a local directory,/home/student/.kube/cache/discovery/10.128.0.3_6443. If you cannot find the
    lines, you may want to redirect all output to a file andgrepfor them.

```
student@lfs458-node-1a0a:~$ strace kubectl get endpoints
execve("/usr/bin/kubectl", ["kubectl", "get", "endpoints"], [/*....
....
openat(AT_FDCWD, "/home/student/.kube/cache/discovery/10.128.0.3_6443..
<output_omitted>
```
3. Change to the parent directory and explore. Your endpoint IP will be different, so replace the following with one suited
    to your system.

```
student@lfs458-node-1a0a:~$ cd /home/student/.kube/cache/discovery/
```
```
student@lfs458-node-1a0a:~/.kube/cache/discovery$ ls
10.128.0.3_6443
```
```
student@lfs458-node-1a0a:~/.kube/cache/discovery$ cd 10.128.0.3_6443/
```
4. View the contents. You will find there are directories with various configuration information for kubernetes.


#### 34 CHAPTER 5. APIS AND ACCESS

```
student@lfs458-node-1a0a:~/.kube/cache/discovery/10.128.0.3_6443$ ls
admissionregistration.k8s.io batch policy
apiextensions.k8s.io certificates.k8s.io rbac.authorization.k8s.io
apiregistration.k8s.io coordination.k8s.io scheduling.k8s.io
apps crd.projectcalico.org servergroups.json
authentication.k8s.io events.k8s.io storage.k8s.io
authorization.k8s.io extensions v1
autoscaling networking.k8s.io
```
5. Use the find command to list out the subfiles. The prompt has been modified to look better on this page.

```
student@lfs458-node-1a0a:./10.128.0.3_6443$ find.
.
./events.k8s.io
./events.k8s.io/v1beta1
./events.k8s.io/v1beta1/serverresources.json
./apps
./apps/v1
./apps/v1/serverresources.json
./apps/v1beta1
./apps/v1beta1/serverresources.json
<output_omitted>
```
6. View the objects available in version 1 of the API. For each object, or kind:, you can view the verbs or actions for that
    object, such as create seen in the following example. Note the prompt has been truncated for the command to fit on one
    line. Some are HTTP verbs, such asGET, others are product specific options, not standard HTTP verbs.

```
student@lfs458-node-1a0a:.$ python -m json.tool v1/serverresources.json
{
"apiVersion": "v1",
"groupVersion": "v1",
"kind": "APIResourceList",
"resources": [
{
"kind": "Binding",
"name": "bindings",
"namespaced": true,
"singularName": "",
"verbs": [
"create"
]
},
<output_omitted>
```
7. Some of the objects haveshortNames, which makes using them on the command line much easier. Locate the
    shortNamefor endpoints.

```
student@lfs458-node-1a0a:.$ python -m json.tool v1/serverresources.json | less
.
{
"kind": "Endpoints",
"name": "endpoints",
"namespaced": true,
"shortNames": [
"ep"
],
"singularName": "",
"verbs": [
"create",
"delete",
.
```
8. Use theshortNameto view the endpoints. It should match the output from the previous command.


#### 5.1. LABS 35

```
student@lfs458-node-1a0a:.$ kubectl get ep
NAME ENDPOINTS AGE
kubernetes 10.128.0.3:6443 3h
```
9. We can see there are 37 objects in version 1 file.

```
student@lfs458-node-1a0a:.$ python -m json.tool v1/serverresources.json | grep kind
"kind": "APIResourceList",
"kind": "Binding",
"kind": "ComponentStatus",
"kind": "ConfigMap",
"kind": "Endpoints",
"kind": "Event",
<output_omitted>
```
10. Looking at another file we find nine more.

```
student@lfs458-node-1a0a:$ python -m json.tool \
apps/v1beta1/serverresources.json | grep kind
"kind": "APIResourceList",
"kind": "ControllerRevision",
"kind": "Deployment",
<output_omitted>
```
11. Delete thecurlpodto recoup system resources.

```
student@lfs458-node-1a0a:$ kubectl delete po curlpod
pod "curlpod" deleted
```
12. Take a look around the other files in this directory as time permits.


#### 36 CHAPTER 5. APIS AND ACCESS


**Chapter 6**

**API Objects**

### 6.1 Labs

### Exercise 6.1: RESTful API Access

### Overview

We will continue to explore ways of accessing the control plane of our cluster. In the security chapter we will discuss there
are several authentication methods, one of which is use of aBearer tokenWe will work with one then deploy a local proxy
server for application-level access to the Kubernetes API.

### RESTful API Access

We will use the **curl** command to make API requests to the cluster, in an in-secure manner. Once we know the IP address
and port, then the token we can retrieve cluster data in a RESTful manner. By default most of the information is restricted, but
changes to authentication policy could allow more access.

1. First we need to know the IP and port of a node running a replica of the API server. The master system will typically
    have one running. Use **kubectl config view** to get overall cluster configuration, and find the server entry. This will give
    us both the IP and the port.

```
student@lfs458-node-1a0a:~$ kubectl config view
apiVersion: v1
clusters:
```
- cluster:
    certificate-authority-data: REDACTED
    server: https://10.128.0.3:6443
name: kubernetes
<output_omitted>
2. Next we need to find the bearer token. This is part of a default token. Look at a list of tokens, first all on the cluster, then
just those in the default namespace. There will be asecretfor each of the controllers of the cluster.

#### 37


#### 38 CHAPTER 6. API OBJECTS

```
student@lfs458-node-1a0a:~$ kubectl get secrets --all-namespaces
NAMESPACE NAME TYPE ...
default default-token-jdqp7 kubernetes.io/service-account-token...
kube-public default-token-b2prn kubernetes.io/service-account-token...
kube-system attachdetach-controller-token-ckwvh kubernetes.io/servic...
kube-system bootstrap-signer-token-wpx66 kubernetes.io/service-accou...
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ kubectl get secrets
NAME TYPE DATA AGE
default-token-jdqp7 kubernetes.io/service-account-token 3 2d
```
3. Look at the details of the secret. We will need thetoken:information from the output.

```
student@lfs458-node-1a0a:~$ kubectl describe secret default-token-jdqp7
Name: default-token-jdqp7
Namespace: default
Labels: <none>
<output_omitted>
token: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVz
L3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3Bh
Y2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubm
<output_omitted>
```
4. Using your mouse to cut and paste, or **cut** , or **awk** to save the data, from the first charactereyJhto the last,EFmBWAto
    a variable namedtoken. Your token data will be different.

```
student@lfs458-node-1a0a:~$ export token=$(kubectl describe \
secret default-token-jdqp7 |grep ^token |cut -f7 -d ’ ’)
```
5. Test to see if you can get basic API information from your cluster. We will pass it the server name and port, the token
    and use the **-k** option to avoid using a cert.

```
student@lfs458-node-1a0a:~$ curl https://10.128.0.3:6443/apis \
--header "Authorization: Bearer $token" -k
{
"kind": "APIVersions",
"versions": [
"v1"
],
"serverAddressByClientCIDRs": [
{
"clientCIDR": "0.0.0.0/0",
"serverAddress": "10.128.0.3:6443"
}
]
}
<output_omitted>
```
6. Try the same command, but look at API v1.

```
student@lfs458-node-1a0a:~$ curl https://10.128.0.3:6443/api/v1 \
--header "Authorization: Bearer $token" -k
<output_omitted>
```
7. Now try to get a list of namespaces. This should return an error. It shows our request is being seen as
    system:serviceaccount, which does not have theRBACauthorization to list all namespaces in the cluster.

```
student@lfs458-node-1a0a:~$ curl \
https://10.128.0.3:6443/api/v1/namespaces \
--header "Authorization: Bearer $token" -k
<output_omitted>
"message": "namespaces is forbidden: User \"system:serviceaccount:default...
<output_omitted>
```

#### 6.1. LABS 39

8. Pods can also make use of included certificates to use the API. The certificates are automatically made available to
    a pod under the/var/run/secrets/kubernetes.io/serviceaccount/. We will deploy a simple Pod and view the
    resources. If you view thetokenfile you will find it is the same value we put into the$tokenvariable. The **-i** will request
    a **-t** terminal session of thebusyboxcontainer. Once you exit the container will not restart and the pod will show as
    completed.

```
student@lfs458-node-1a0a:~$ kubectl run -i -t busybox --image=busybox \
--restart=Never
# ls /var/run/secrets/kubernetes.io/serviceaccount/
ca.crt namespace token
/ # exit
```
### Exercise 6.2: Using the Proxy

Another way to interact with the API is via aproxy. The proxy can be run from a node or from within aPodthrough the use of
a sidecar. In the following steps we will deploy a proxy listening to the loopback address. We will use **curl** to access the API
server. If the **curl** request works, but does not from outside the cluster, we have narrowed down the issue to authentication
and authorization instead of issues further along the API ingestion process.

1. Begin by starting the proxy. It will start in the foreground by default. There are several options you could pass. Begin by
    reviewing the help output.

```
student@lfs458-node-1a0a:~$ kubectl proxy -h
Creates a proxy server or application-level gateway between localhost
and the Kubernetes API Server. It also allows serving static content
over specified HTTP path. All incoming data enters through one port
and gets forwarded to the remote kubernetes API Server port, except
for the path matching the static content path.
```
```
Examples:
# To proxy all of the kubernetes api and nothing else, use:
```
```
$ kubectl proxy --api-prefix=/
<output_omitted>
```
2. Start the proxy while setting the API prefix, and put it in the background. You may need to useenterto view the prompt.

```
student@lfs458-node-1a0a:~$ kubectl proxy --api-prefix=/ &
[1] 22500
Starting to serve on 127.0.0.1:8001
```
3. Now use the same **curl** command, but point toward the IP and port shown by the proxy. The output should be the same
    as without the proxy, but may be formatted differently.

```
student@lfs458-node-1a0a:~$ curl http://127.0.0.1:8001/api/
<output_omitted>
```
4. Make an API call to retrieve the namespaces. The command did not work in the previous section due to permissions,
    but should work now as theproxyis making the request on your behalf.

```
student@lfs458-node-1a0a:~$ curl http://127.0.0.1:8001/api/v1/namespaces
{
"kind": "NamespaceList",
"apiVersion": "v1",
"metadata": {
"selfLink": "/api/v1/namespaces",
"resourceVersion": "86902"
<output_omitted>
```
### Exercise 6.3: Working with Jobs

While most API objects are deployed such that they continue to be available there are some which we may want to run a
particular number of times called aJob, and others on a regular basis called aCronJob


#### 40 CHAPTER 6. API OBJECTS

**Create A Job**

1. Create a job which will run a container which sleeps for three seconds then stops.

```
student@lfs458-node-1a0a:~$ vim job.yaml
apiVersion: batch/v1
kind: Job
metadata:
name: sleepy
spec:
template:
spec:
containers:
```
- name: resting
    image: busybox
    command: ["/bin/sleep"]
    args: ["3"]
restartPolicy: Never
2. Create the job, then verify and view the details. The example shows checking the job three seconds in and then again
after it has completed. You may see different output depending on how fast you type.

```
student@lfs458-node-1a0a:~$ kubectl create -f job.yaml
job.batch/sleepy created
```
```
student@lfs458-node-1a0a:~$ kubectl get job
NAME COMPLETIONS DURATION AGE
sleepy 0/1 3s 3s
```
```
student@lfs458-node-1a0a:~$ kubectl describe jobs.batch sleepy
Name: sleepy
Namespace: default
Selector: controller-uid=24c91245-d0fb-11e8-947a-42010a800002
Labels: controller-uid=24c91245-d0fb-11e8-947a-42010a800002
job-name=sleepy
Annotations: <none>
Parallelism: 1
Completions: 1
Start Time: Tue, 16 Oct 2018 04:22:50 +0000
Completed At: Tue, 16 Oct 2018 04:22:55 +0000
Duration: 5s
Pods Statuses: 0 Running / 1 Succeeded / 0 Failed
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ kubectl get job
NAME COMPLETIONS DURATION AGE
sleepy 1/1 5s 17s
```
3. View the configuration information of the job. There are three parameters we can use to affect how the job runs. Use **-o**
    **yaml** to see these parameters. We can see thatbackoffLimit,completions, and theparallelism. We’ll add these
    parameters next.

```
student@lfs458-node-1a0a:~$ kubectl get jobs.batch sleepy -o yaml
```
```
<output_omitted>
uid: c2c3a80d-d0fc-11e8-947a-42010a800002
spec:
backoffLimit: 6
completions: 1
parallelism: 1
selector:
matchLabels:
<output_omitted>
```

#### 6.1. LABS 41

4. As the job continues toAGEin a completion state, delete the job.

```
student@lfs458-node-1a0a:~$ kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted
```
5. Edit the YAML and add thecompletions:parameter and set it to **5**.

```
student@lfs458-node-1a0a:~$ vim job.yaml
<output_omitted>
metadata:
name: sleepy
spec:
completions: 5 #<--Add this line
template:
spec:
containers:
<output_omitted>
```
6. Create the job again. As you view the job note thatCOMPLETIONSbegins as zero of **5**.

```
student@lfs458-node-1a0a:~$ kubectl create -f job.yaml
job.batch/sleepy created
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs.batch
NAME COMPLETIONS DURATION AGE
sleepy 0/5 5s 5s
```
7. View the pods that running. Again the output may be different depending on the speed of typing.

```
student@lfs458-node-1a0a:~$ kubectl get pods
NAME READY STATUS RESTARTS AGE
sleepy-z5tnh 0/1 Completed 0 8s
sleepy-zd692 1/1 Running 0 3s
<output_omitted>
```
8. Eventually all the jobs will have completed. Verify then delete the job.

```
student@lfs458-node-1a0a:~$ kubectl get jobs
NAME COMPLETIONS DURATION AGE
sleepy 5/5 26s 10m
```
```
student@lfs458-node-1a0a:~$ kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted
```
9. Edit the YAML again. This time add in theparallelism:parameter. Set it to **2** such that two pods at a time will be
    deployed.

```
student@lfs458-node-1a0a:~$ job job.yaml
<output_omitted>
name: sleepy
spec:
completions: 5
parallelism: 2 #<-- Add this line
template:
spec:
<output_omitted>
```
10. Create thejobagain. You should see the pods deployed two at a time until all five have completed.

```
student@lfs458-node-1a0a:~$ kubectl create -f job.yaml
job.batch/sleepy created
```
```
student@lfs458-node-1a0a:~$ kubectl get pods
NAME READY STATUS RESTARTS AGE
```

#### 42 CHAPTER 6. API OBJECTS

```
sleepy-8xwpc 1/1 Running 0 5s
sleepy-xjqnf 1/1 Running 0 5s
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs
NAME COMPLETIONS DURATION AGE
sleepy 3/5 11s 11s
```
11. Add a parameter which will stop the job after a certain number of seconds. Set theactiveDeadlineSeconds:to 15.
    The job and all pods will end once it runs for 15 seconds. We will also increase the sleep argument to five, just to be
    sure does not expire by itself.

```
student@lfs458-node-1a0a:~$ vim job.yaml
<output_omitted>
completions: 5
parallelism: 2
activeDeadlineSeconds: 15 #<-- Add this line
template:
spec:
containers:
```
- name: resting
    image: busybox
    command: ["/bin/sleep"]
    args: ["5"] #<-- Edit this line
<output_omitted>
12. Delete and recreate the job again. It should run for 15 seconds, usually 3/5, then continue to age without further
completions.

```
student@lfs458-node-1a0a:~$ kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl create -f job.yaml
job.batch/sleepy created
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs
NAME COMPLETIONS DURATION AGE
sleepy 1/5 6s 6s
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs
NAME COMPLETIONS DURATION AGE
sleepy 3/5 16s 16s
```
13. View themessage:entry in theStatussection of the object YAML output.

```
student@lfs458-node-1a0a:~$ kubectl get job sleepy -o yaml
<output_omitted>
status:
conditions:
```
- lastProbeTime: 2018-10-16T05:45:14Z
    lastTransitionTime: 2018-10-16T05:45:14Z
    message: Job was active longer than specified deadline
    reason: DeadlineExceeded
    status: "True"
    type: Failed
failed: 2
startTime: 2018-10-16T05:44:59Z
succeeded: 3
14. Delete the job.

```
student@lfs458-node-1a0a:~$ kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted
```

#### 6.1. LABS 43

**Create a CronJob**

ACronJobcreates a watch loop which will create a batch job on your behalf when the time becomes true. We Will use our
existingJobfile to start.

1. Copy theJobfile to a new file.

```
student@lfs458-node-1a0a:~$ cp job.yaml cronjob.yaml
```
2. Edit the file to look like the annotated file shown below. Edit the lines mentioned below. The three parameters we added
    will need to be removed. Other lines will need to be further indented.

```
student@lfs458-node-1a0a:~$ vim cronjob.yaml
apiVersion: batch/v1beta1 #<-- Add beta1 to be v1beta1
kind: CronJob #<-- Update this line to CronJob
metadata:
name: sleepy
spec:
schedule: "*/2 * * * *" #<-- Add Linux style cronjob syntax
jobTemplate: #<-- New jobTemplate and spec move
spec:
template: #<-- This and following lines move
spec: #<-- four spaces to the right
containers:
```
- name: resting
    image: busybox
    command: ["/bin/sleep"]
    args: ["3"]
restartPolicy: Never
3. Create the newCronJob. View the jobs. It will take two minutes for theCronJobto run and generate a new batchJob.

```
student@lfs458-node-1a0a:~$ kubectl create -f cronjob.yaml
cronjob.batch/sleepy created
```
```
student@lfs458-node-1a0a:~$ kubectl get cronjobs.batch
NAME SCHEDULE SUSPEND ACTIVE LAST SCHEDULE AGE
sleepy */2 * * * * False 0 <none> 8s
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs.batch
No resources found.
```
4. After two minutes you should see jobs start to run.

```
student@lfs458-node-1a0a:~$ kubectl get cronjobs.batch
NAME SCHEDULE SUSPEND ACTIVE LAST SCHEDULE AGE
sleepy */2 * * * * False 0 21s 2m1s
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs.batch
NAME COMPLETIONS DURATION AGE
sleepy-1539722040 1/1 5s 18s
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs.batch
NAME COMPLETIONS DURATION AGE
sleepy-1539722040 1/1 5s 5m17s
sleepy-1539722160 1/1 6s 3m17s
sleepy-1539722280 1/1 6s 77s
```
5. Ensure that if the job continues for more than 10 seconds it is terminated. We will first edit the **sleep** command to run
    for 30 seconds then add theactiveDeadlineSeconds:entry to the container.


#### 44 CHAPTER 6. API OBJECTS

```
student@lfs458-node-1a0a:~$ vim cronjob.yaml
....
jobTemplate:
spec:
template:
spec:
activeDeadlineSeconds: 10 #<-- Add this line
containers:
```
- name: resting
....
6. Delete and recreate theCronJob. It may take a couple of minutes for the batchJobto be created and terminate due to
the timer.

```
student@lfs458-node-1a0a:~$ kubectl delete cronjobs.batch sleepy
cronjob.batch "sleepy" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl create -f cronjob.yaml
cronjob.batch/sleepy created
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs
NAME COMPLETIONS DURATION AGE
sleepy-1539723240 0/1 61s 61s
```
```
student@lfs458-node-1a0a:~$ kubectl get cronjobs.batch
NAME SCHEDULE SUSPEND ACTIVE LAST SCHEDULE AGE
sleepy */2 * * * * False 1 72s 94s
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs
NAME COMPLETIONS DURATION AGE
sleepy-1539723240 0/1 75s 75s
```
```
student@lfs458-node-1a0a:~$ kubectl get jobs
NAME COMPLETIONS DURATION AGE
sleepy-1539723240 0/1 2m19s 2m19s
sleepy-1539723360 0/1 19s 19s
```
```
student@lfs458-node-1a0a:~$ kubectl get cronjobs.batch
NAME SCHEDULE SUSPEND ACTIVE LAST SCHEDULE AGE
sleepy */2 * * * * False 2 31s 2m53s
```
7. Clean up by deleting theCronJob.

```
student@lfs458-node-1a0a:~$ kubectl delete cronjobs.batch sleepy
cronjob.batch "sleepy" deleted
```

**Chapter 7**

**Managing State With Deployments**

### 7.1 Labs

### Exercise 7.1: Working with ReplicaSets

### Overview

Understanding and managing the state of containers is a core Kubernetes task. In this lab we will first explore the API objects
used to manage groups of containers. The objects available have changed as Kubernetes has matured, so the Kubernetes
version in use will determine which are available. Our first object will be aReplicaSet, which does not include newer
management features found withDeployments. ADeploymentwill will manageReplicaSetsfor you. We will also work with
another object called aDaemonSetwhich ensures a container is running on newly added node.

Then we will update the software in a container, view the revision history, and roll-back to a previous version.

### Working with ReplicaSets

AReplicaSetis a next-generation of aReplication Controller, which differs only in the selectors supported. The only
reason to use aReplicaSetanymore is if you have no need for updating container software or require update orchestration
which won’t work with the typical process.

1. View any currentReplicaSets. If you deleted resources at the end of a previous lab, you should have none reported in
    thedefaultnamespace.

```
student@lfs458-node-1a0a:~$ kubectl get rs
No resources found.
```
2. Create a YAML file for a simpleReplicaSet. TheapiVersionsetting depends on the version of Kubernetes you are
    using. Versions 1.8 and beyond will useapps/v1beta1, then perhaps somedayapps/v1beta2and then probably a
    stableapps/v1. We will use an older version of **nginx** then update to a newer version later in the exercise.

```
student@lfs458-node-1a0a:~$ vim rs.yaml
```
#### 45


#### 46 CHAPTER 7. MANAGING STATE WITH DEPLOYMENTS

```
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
name: rs-one
spec:
replicas: 2
template:
metadata:
labels:
system: ReplicaOne
spec:
containers:
```
- name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
3. Create theReplicaSet:

```
student@lfs458-node-1a0a:~$ kubectl create -f rs.yaml
replicaset.extensions/rs-one created
```
4. View the newly createdReplicaSet:

```
student@lfs458-node-1a0a:~$ kubectl describe rs rs-one
Name: rs-one
Namespace: default
Selector: system=ReplicaOne
Labels: system=ReplicaOne
Annotations: <none>
Replicas: 2 current / 2 desired
Pods Status: 2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
Labels: system=ReplicaOne
Containers:
nginx:
Image: nginx:1.7.9
Port: 80/TCP
Environment: <none>
Mounts: <none>
Volumes: <none>
Events: <none>
```
5. View the Pods created with theReplicaSet. From the yaml file created there should be two Pods. You may see a
    Completedbusybox which will be cleared out eventually.

```
student@lfs458-node-1a0a:~$ kubectl get pods
NAME READY STATUS RESTARTS AGE
rs-one-2p9x4 1/1 Running 0 5m4s
rs-one-3c6pb 1/1 Running 0 5m4s
```
6. Now we will delete theReplicaSet, but not the Pods it controls.

```
student@lfs458-node-1a0a:~$ kubectl delete rs rs-one --cascade=false
replicaset.extensions "rs-one" deleted
```
```
View theReplicaSetand Pods again:
7.student@lfs458-node-1a0a:~$ kubectl describe rs rs-one
Error from server (NotFound): replicasets.extensions "rs-one" not found
```
```
student@lfs458-node-1a0a:~$ kubectl get pods
```

#### 7.1. LABS 47

#### NAME READY STATUS RESTARTS AGE

```
rs-one-2p9x4 1/1 Running 0 7m
rs-one-3c6pb 1/1 Running 0 7m
```
8. Create theReplicaSetagain. As long as we do not change theselectorfield, the newReplicaSetshould take
    ownership. Pod software versions cannot be updated this way.

```
student@lfs458-node-1a0a:~$ kubectl create -f rs.yaml
replicaset.extensions/rs-one created
```
9. View the age of theReplicaSetand then the Pods within:

```
student@lfs458-node-1a0a:~$ kubectl get rs
NAME DESIRED CURRENT READY AGE
rs-one 2 2 2 46s
```
```
student@lfs458-node-1a0a:~$ kubectl get pods
NAME READY STATUS RESTARTS AGE
rs-one-2p9x4 1/1 Running 0 8m
rs-one-3c6pb 1/1 Running 0 8m
```
10. We will now isolate a Pod from itsReplicaSet. Begin by editing the label of a Pod. We will change thesystem:
    parameter to beIsolatedPod.

```
student@lfs458-node-1a0a:~$ kubectl edit po rs-one-3c6pb
```
```
....
labels:
system: IsolatedPod #<-- Change from ReplicaOne
name: rs-one-3c6pb
....
```
11. View the number of pods within theReplicaSet. You should see two running.

```
student@lfs458-node-1a0a:~$ kubectl get rs
NAME DESIRED CURRENT READY AGE
rs-one 2 2 2 4m
```
12. Now view the pods with the label key ofsystem. You should note that there are three, with one being newer than others.
    TheReplicaSetmade sure to keep two replicas, replacing the Pod which was isolated.

```
student@lfs458-node-1a0a:~$ kubectl get po -L system
NAME READY STATUS RESTARTS AGE SYSTEM
rs-one-3c6pb 1/1 Running 0 10m IsolatedPod
rs-one-2p9x4 1/1 Running 0 10m ReplicaOne
rs-one-dq5xd 1/1 Running 0 30s ReplicaOne
```
13. Delete theReplicaSet, then view any remaining Pods.

```
student@lfs458-node-1a0a:~$ kubectl delete rs rs-one
replicaset.extensions "rs-one" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
rs-one-3c6pb 1/1 Running 0 14m
rs-one-dq5xd 0/1 Terminating 0 4m
```
14. In the above example the Pods had not finished termination. Wait for a bit and check again. There should be no
    ReplicaSets, but one Pod.


#### 48 CHAPTER 7. MANAGING STATE WITH DEPLOYMENTS

```
student@lfs458-node-1a0a:~$ kubectl get rs
No resources found.
```
```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
rs-one-3c6pb 1/1 Running 0 16m
```
15. Delete the remaining Pod using the label.

```
student@lfs458-node-1a0a:~$ kubectl delete po -l system=IsolatedPod
pod "rs-one-3c6pb" deleted
```
### Exercise 7.2: Working with DaemonSets

ADaemonSetis a watch loop object like aDeploymentwhich we have been working with in the rest of the labs. TheDaemonSet
ensures that when a node is added to a cluster a pods will be created on that node. ADeploymentwould only ensure a
particular number of pods are created in general, several could be on a single node. Using aDaemonSetcan be helpful to
ensure applications are on each node, helpful for things like metrics and logging especially in large clusters where hardware
my be swapped out often. Should a node be be removed from a cluster theDaemonSetwould ensure the Pods are garbage
collected before removal. Starting with Kubernetes v1.12 the scheduler handlesDaemonSetdeployment which means we can
now configure certain nodes to not have a particularDaemonSetpods.

This extra step of automation can be useful for using with products like **ceph** where storage is often added or removed, but
perhaps among a subset of hardware. They allow for complex deployments when used with declared resources like memory,
CPU or volumes.

1. We begin by creating a yaml file. In this case thekindwould be set toDaemonSet. For ease of use we will copy the
    previously createdrs.yamlfile and make a couple edits. Remove theReplicas: 2line.

```
student@lfs458-node-1a0a:~$ cp rs.yaml ds.yaml
```
```
student@lfs458-node-1a0a:~$ vim ds.yaml
....
kind: DaemonSet
....
name: ds-one
....
replicas: 2 #<<<----Remove this line
....
system: DaemonSetOne
....
```
2. Create and verify the newly formed DaemonSet. There should be one Pod per node in the cluster.

```
student@lfs458-node-1a0a:~$ kubectl create -f ds.yaml
daemonset.extensions/ds-one created
```
```
student@lfs458-node-1a0a:~$ kubectl get ds
NAME DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE-SELECTOR AGE
ds-one 2 2 2 2 2 <none> 1m
```
```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
ds-one-b1dcv 1/1 Running 0 2m
ds-one-z31r4 1/1 Running 0 2m
```
3. Verify the image running inside the Pods. We will use this information in the next section.

```
student@lfs458-node-1a0a:~$ kubectl describe po ds-one-b1dcv | grep Image:
Image: nginx:1.7.9
```

#### 7.1. LABS 49

### Exercise 7.3: Rolling Updates and Rollbacks

One of the advantages of micro-services is the ability to replace and upgrade a container while continuing to respond to client
requests. We will use the defaultOnDeletesetting that upgrades a container when the predecessor is deleted, then the use
theRollingUpdatefeature as well.

1. Begin by viewing the currentupdateStrategysetting for theDaemonSetcreated in the previous section.

```
student@lfs458-node-1a0a:~$ kubectl get ds ds-one -o yaml \
| grep -A 1 Strategy
updateStrategy:
type: OnDelete
```
2. Update the DaemonSet to use a newer version of the **nginx** server. This time use the **set** command instead of **edit**. Set
    the version to be1.8.1-alpine.

```
student@lfs458-node-1a0a:~$ kubectl set image ds ds-one nginx=nginx:1.8.1-alpine
daemonset.extensions/ds-one image updated
```
3. Verify that theImage:parameter for the Pod checked in the previous section is unchanged.

```
student@lfs458-node-1a0a:~$ kubectl describe po ds-one-b1dcv |grep Image:
Image: nginx:1.7.9
```
4. Delete the Pod. Wait until the replacement Pod is running and check the version.

```
student@lfs458-node-1a0a:~$ kubectl delete po ds-one-b1dcv
pod "ds-one-b1dcv" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
ds-one-xc86w 1/1 Running 0 19s
ds-one-z31r4 1/1 Running 0 4m8s
```
```
student@lfs458-node-1a0a:~$ kubectl describe po ds-one-xc86w |grep Image:
Image: nginx:1.8.1-alpine
```
5. View the image running on the older Pod. It should still show version 1.7.9.

```
student@lfs458-node-1a0a:~$ kubectl describe po ds-one-z31r4 |grep Image:
Image: nginx:1.7.9
```
6. View the history of changes for theDaemonSet. You should see two revisions listed. The number of revisions kept is set
    in theDaemonSetwith v.1.12.1 the history kept has increased to ten from two, by default.

```
student@lfs458-node-1a0a:~$ kubectl rollout history ds ds-one
daemonsets "ds-one"
REVISION CHANGE-CAUSE
1 <none>
2 <none>
```
7. View the settings for the various versions of theDaemonSet. TheImage:line should be the only difference between the
    two outputs.

```
student@lfs458-node-1a0a:~$ kubectl rollout history ds ds-one --revision=1
daemonsets "ds-one" with revision #1
Pod Template:
Labels: system=DaemonSetOne
Containers:
nginx:
Image: nginx:1.7.9
Port: 80/TCP
Environment: <none>
```

#### 50 CHAPTER 7. MANAGING STATE WITH DEPLOYMENTS

```
Mounts: <none>
Volumes: <none>
```
```
student@lfs458-node-1a0a:~$ kubectl rollout history ds ds-one --revision=2
....
Image: nginx:1.8.1-alpine
.....
```
8. Usekubectl rollout undoto change theDaemonSetback to an earlier version. As we are still using theOnDelete
    strategy there should be no change to the Pods.

```
student@lfs458-node-1a0a:~$ kubectl rollout undo ds ds-one --to-revision=1
daemonset.extensions/ds-one rolled back
```
```
student@lfs458-node-1a0a:~$ kubectl describe po ds-one-xc86w |grep Image:
Image: nginx:1.8.1-alpine
```
9. Delete the Pod, wait for the replacement to spawn then check the image version again.

```
student@lfs458-node-1a0a:~$ kubectl delete po ds-one-xc86w
pod "ds-one-xc86w" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
ds-one-qc72k 1/1 Running 0 10s
ds-one-xc86w 0/1 Terminating 0 12m
ds-one-z31r4 1/1 Running 0 28m
```
```
student@lfs458-node-1a0a:~$ kubectl describe po ds-one-qc72k |grep Image:
Image: nginx:1.7.9
```
10. View the details of theDaemonSet. The Image should be v1.7.9 in the output.

```
student@lfs458-node-1a0a:~$ kubectl describe ds |grep Image:
Image: nginx:1.7.9
```
11. View the current configuration for theDaemonSetin YAML output. Look for the update strategy near the end of the
    output.

```
student@lfs458-node-1a0a:~$ kubectl get ds ds-one -o yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
.....
terminationGracePeriodSeconds: 30
templateGeneration: 3
updateStrategy:
type: OnDelete
status:
currentNumberScheduled: 2
.....
```
12. Create a newDaemonSet, this time setting the update policy toRollingUpdate. Begin by generating a new config file.

```
student@lfs458-node-1a0a:~$ kubectl get ds ds-one -o yaml --export > ds2.yaml
```
13. Edit the file. Change the name, around line eight and the update strategy around line 38.

```
student@lfs458-node-1a0a:~$ vim ds2.yaml
....
name: ds-two
....
type: RollingUpdate
```

#### 7.1. LABS 51

14. Create the newDaemonSetand verify the **nginx** version in the new pods.

```
student@lfs458-node-1a0a:~$ kubectl create -f ds2.yaml
daemonset.extensions/ds-two created
```
```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
ds-one-qc72k 1/1 Running 0 28m
ds-one-z31r4 1/1 Running 0 57m
ds-two-10khc 1/1 Running 0 5m
ds-two-kzp9g 1/1 Running 0 5m
```
```
student@lfs458-node-1a0a:~$ kubectl describe po ds-two-10khc |grep Image:
Image: nginx:1.7.9
```
15. Edit the configuration file and set the image to a newer version such as 1.8.1-alpine.

```
student@lfs458-node-1a0a:~$ kubectl edit ds ds-two
....
```
- image: nginx:1.8.1-alpine
.....
16. View the age of theDaemonSets. It should be around ten minutes old, depending on how fast you type.

```
student@lfs458-node-1a0a:~$ kubectl get ds ds-two
NAME DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE-SELECTOR AGE
ds-two 2 2 2 2 2 <none> 10m
```
17. Now view the age of the Pods. Two should be much younger than theDaemonSet. They are also a few seconds apart
    due to the nature of the rolling update where one then the other pod was terminated and recreated.

```
student@lfs458-node-1a0a:~$ kubectl get po
NAME READY STATUS RESTARTS AGE
ds-one-qc72k 1/1 Running 0 36m
ds-one-z31r4 1/1 Running 0 1h
ds-two-2p8vz 1/1 Running 0 34s
ds-two-8lx7k 1/1 Running 0 32s
```
18. Verify the Pods are using the new version of the software.

```
student@lfs458-node-1a0a:~$ kubectl describe po ds-two-8lx7k |grep Image:
Image: nginx:1.8.1-alpine
```
19. View the rollout status and the history of theDaemonSets.

```
student@lfs458-node-1a0a:~$ kubectl rollout status ds ds-two
daemon set "ds-two" successfully rolled out
```
```
student@lfs458-node-1a0a:~$ kubectl rollout history ds ds-two
daemonsets "ds-two"
REVISION CHANGE-CAUSE
1 <none>
2 <none>
```
20. View the changes in the update they should look the same as the previous history, but did not require the Pods to be
    deleted for the update to take place.

```
student@lfs458-node-1a0a:~$ kubectl rollout history ds ds-two --revision=2
...
Image: nginx:1.8.1-alpine
```
21. Clean up the system by removing one of theDaemonSets. We will leave the other running.

```
student@lfs458-node-1a0a:~$ kubectl delete ds ds-two
daemonset.extensions "ds-two" deleted
```

#### 52 CHAPTER 7. MANAGING STATE WITH DEPLOYMENTS


**Chapter 8**

**Services**

### 8.1 Labs

### Exercise 8.1: Deploy A New Service

### Overview

**Services** (also called **microservices** ) are objects which declare a policy to access a logical set of Pods. They are typically
assigned withlabelsto allow persistent access to a resource, when front or back end containers are terminated and replaced.

Native applications can use theEndpointsAPI for access. Non-native applications can use a Virtual IP-based bridge to
access back end pods.ServiceTypes Typecould be:

- **ClusterIP** default - exposes on a cluster-internal IP. Only reachable within cluster
- **NodePort** Exposes node IP at a static port. A ClusterIP is also automatically created.
- **LoadBalancer** Exposes service externally using cloud providers load balancer.NodePortandClusterIPautomatically
    created.
- **ExternalName** Maps service to contents ofexternalNameusing aCNAMErecord.

We use services as part of decoupling such that any agent or object can be replaced without interruption to access from client
to back end application.

### Deploy A New Service

1. Deploy two **nginx** servers using **kubectl** and a new.yamlfile. We will use thev1betaversion of the API. The kind
    should beDeploymentand label it withnginx. Create two replicas and expose port 8080. What follows is a well
    documented file. There is no need to include the comments when you create the file. This file can also be found among
    the other examples in the tarball.

```
student@lfs458-node-1a0a:~$ vim nginx-one.yaml
```
#### 53


#### 54 CHAPTER 8. SERVICES

```
apiVersion: extensions/v1beta1
# Determines YAML versioned schema.
kind: Deployment
# Describes the resource defined in this file.
metadata:
name: nginx-one
labels:
system: secondary
# Required string which defines object within namespace.
namespace: accounting
# Existing namespace resource will be deployed into.
spec:
replicas: 2
# How many Pods of following containers to deploy
template:
metadata:
labels:
app: nginx
# Some string meaningful to users, not cluster. Keys
# must be unique for each object. Allows for mapping
# to customer needs.
spec:
containers:
# Array of objects describing containerized application with a Pod.
# Referenced with shorthand spec.template.spec.containers
```
- image: nginx:1.7.9
# The Docker image to deploy
imagePullPolicy: Always
name: nginx
# Unique name for each container, use local or Docker repo image
ports:
- containerPort: 8080
protocol: TCP
# Optional resources this container may need to function.
nodeSelector:
system: secondOne
# One method of node affinity.
2. View the existing labels on the nodes in the cluster.

```
student@lfs458-node-1a0a:~$ kubectl get nodes --show-labels
<output_omitted>
```
3. Run the following command and look for the errors. Assuming there is no typo, you should have gotten an error about
    about theaccountingnamespace.

```
student@lfs458-node-1a0a:~$ kubectl create -f nginx-one.yaml
Error from server (NotFound): error when creating
"nginx-one.yaml": namespaces "accounting" not found
```
4. Create the namespace and try to create the deployment again. There should be no errors this time.

```
student@lfs458-node-1a0a:~$ kubectl create ns accounting
namespace/accounting" created
```
```
student@lfs458-node-1a0a:~$ kubectl create -f nginx-one.yaml
deployment.extensions/nginx-one created
```
5. View the status of the new nodes. Note they do not show aRunningstatus.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting get pods
NAME READY STATUS RESTARTS AGE
nginx-one-74dd9d578d-fcpmv 0/1 Pending 0 4m
nginx-one-74dd9d578d-r2d67 0/1 Pending 0 4m
```

#### 8.1. LABS 55

6. View the node each has been assigned to (or not) and the reason, which shows under events at the end of the output.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting describe pod \
nginx-one-74dd9d578d-fcpmv
```
```
Name: nginx-one-74dd9d578d-fcpmv
Namespace: accounting
Node: <none>
```
```
<output_omitted>
```
```
Events:
Type Reason Age From ....
---- ------ ---- ----
Warning FailedScheduling 37s (x25 over 2m29s) default-scheduler
0/2 nodes are available: 2 node(s) didn’t match node selector.
```
7. Label the secondary node. Verify the labels.

```
student@lfs458-node-1a0a:~$ kubectl label node lfs458-worker \
system=secondOne
node/lfs458-worker labeled
```
```
student@lfs458-node-1a0a:~$ kubectl get nodes --show-labels
NAME STATUS ROLES AGE VERSION LABELS
lfs458-node-1a0a Ready master 1d1h v1.12.1 \
beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
hostname=lfs458-node-1a0a,node-role.kubernetes.io/master=
lfs458-worker Ready <none> 1d1h v1.12.1 \
beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/
hostname=lfs458-worker,system=secondOne
```
8. View the pods in theaccountingnamespace. They may still show asPending. Depending on how long it has been
    since you attempted deployment the system may not have checked for the label. If the Pods showPendingafter a
    minute delete one of the pods. They should both show asRunningafter as a deletion. A change in state will cause the
    Deploymentcontroller to check the status of both Pods.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting get pods
NAME READY STATUS RESTARTS AGE
nginx-one-74dd9d578d-fcpmv 1/1 Running 0 10m
nginx-one-74dd9d578d-sts5l 1/1 Running 0 3s
```
9. View Pods by the label we set in the YAML file. If you look back the Pods were given a label ofapp=nginx.

```
student@lfs458-node-1a0a:~$ kubectl get pods -l app=nginx --all-namespaces
NAMESPACE NAME READY STATUS RESTARTS AGE
accounting nginx-one-74dd9d578d-fcpmv 1/1 Running 0 20m
accounting nginx-one-74dd9d578d-sts5l 1/1 Running 0 9m
```
10. Recall that we exposed port 8080 in the YAML file. Expose the new deployment.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting expose deployment nginx-one
service/nginx-one exposed
```
11. View the newly exposed endpoints. Note that port 8080 has been exposed on each Pod.

```
student@lfs458-node-9q6r:~$ kubectl -n accounting get ep nginx-one
NAME ENDPOINTS AGE
nginx-one 192.168.1.72:8080,192.168.1.73:8080 47s
```
12. Attempt to access the Pod on port 8080, then on port 80. Even though we exposed port 8080 of the container the
    application within has not been configured to listen on this port. The **nginx** server will listens on port 80 by default. A
    curlcommand to that port should return the typical welcome page.


#### 56 CHAPTER 8. SERVICES

```
student@lfs458-node-1a0a:~$ curl 192.168.1.72:8080
curl: (7) Failed to connect to 192.168.1.72 port 8080: Connection refused
```
```
student@lfs458-node-1a0a:~$ curl 192.168.1.72:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<output_omitted>
```
13. Delete the deployment. Edit the YAML file to expose port 80 and create the deployment again.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting delete deploy nginx-one
deployment.extensions "nginx-one" deleted
```
```
student@lfs458-node-1a0a:~$ vim nginx-one.yaml
```
```
student@lfs458-node-1a0a:~$ kubectl create -f nginx-one.yaml
deployment.extensions/nginx-one created
```
### Exercise 8.2: Configure a NodePort

In a previous exercise we deployed aLoadBalancerwhich deployed aClusterIPandNodePortautomatically. In this exercise
we will deploy aNodePort. While you can access a container from within the cluster, one can use aNodePortto NAT traffic
from outside the cluster. One reason to deploy aNodePortinstead, is that aLoadBalanceris also a load balancer resource
from cloud providers like GKE and AWS.

1. In a previous step we were able to view the **nginx** page using the internal Pod IP address. Now expose the deployment
    using the--type=NodePort. We will also give it an easy to remember name and place it in theaccountingnamespace.
    We could pass the port as well, which could help with opening ports in the firewall.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting expose deployment \
nginx-one --type=NodePort --name=service-lab
service/service-lab exposed
```
2. View the details of the services in theaccountingnamespace. We are looking for the autogenerated port.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting describe services
....
NodePort: <unset> 32103/TCP
....
```
3. Locate the exterior facing IP address of the cluster. As we are using GCP nodes, which we access via aFloatingIP,
    we will first check the internal only public IP address. Look for the Kubernetes master URL.

```
student@lfs458-node-1a0a:~$ kubectl cluster-info
Kubernetes master is running at https://10.128.0.3:6443
KubeDNS is running at https://10.128.0.3:6443/api/v1/namespaces/
kube-system/services/kube-dns/proxy
To further debug and diagnose cluster problems, use
’kubectl cluster-info dump’.
```
4. Test access to the **nginx** web server using the combination of master URL andNodePort.

```
student@lfs458-node-1a0a:~$ curl http://10.128.0.3:32103
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

#### 8.1. LABS 57

5. Using the browser on your local system, use the public IP address you use to SSH into your node and the port. You
    should still see the **nginx** default page.

### Exercise 8.3: Use Labels to Manage Resources

1. Try to delete all Pods with theapp=nginxlabel, in all namespaces. You should receive an error as this function must be
    narrowed to a particular namespace. Then delete using the appropriate namespace.

```
student@lfs458-node-1a0a:~$ kubectl delete pods -l app=nginx \
--all-namespaces
Error: unknown flag: --all-namespaces
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ kubectl -n accounting delete pods -l app=nginx
pod "nginx-one-74dd9d578d-fcpmv" deleted
pod "nginx-one-74dd9d578d-sts5l" deleted
```
2. View the Pods again. New versions of the Pods should be running as the controller responsible for them continues.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting get pods
NAME READY STATUS RESTARTS AGE
nginx-one-74dd9d578d-ddt5r 1/1 Running 0 1m
nginx-one-74dd9d578d-hfzml 1/1 Running 0 1m
```
3. We also gave a label to the deployment. View the deployment in the accounting namespace.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting get deploy --show-labels
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE LABELS
nginx-one 2 2 2 2 27m system=secondary
```
4. Delete the deployment using its label.

```
student@lfs458-node-1a0a:~$ kubectl -n accounting delete deploy \
-l system=secondary
deployment.extensions/nginx-one deleted
```
5. Remove the label from the secondary node. Note that the syntax is a minus sign directly after the key you want to
    remove, orsystemin this case.

```
student@lfs458-node-1a0a:~$ kubectl label node lfs458-worker system-
```
```
node/lfs458-worker labeled
```

#### 58 CHAPTER 8. SERVICES


**Chapter 9**

**Volumes and Data**

### 9.1 Labs

### Exercise 9.1: Create a ConfigMap

### Overview

Container files are ephemeral, which can be problematic for some applications. Should a container be restarted the files will
be lost. In addition, we need a method to share files between containers inside a Pod.

AVolumeis a directory accessible to containers in a Pod. Cloud providers offer volumes which persist further than the life of
the Pod, such that AWS or GCE volumes could be pre-populated and offered to Pods, or transferred from one Pod to another.
**Ceph** is also another popular solution for dynamic, persistent volumes.

Unlike current **Docker** volumes a Kubernetes volume has the lifetime of the Pod, not the containers within. You can also use dif-
ferent types of volumes in the same Pod simultaneously, but Volumes cannot mount in a nested fashion. Each must have their
own mount point. Volumes are declared withspec.volumesand mount points withspec.containers.volumeMountspa-
rameters. Each particular volume type, 24 currently, may have other restrictions.https://kubernetes.io/docs/concepts/
storage/volumes/#types-of-volumes

We will also work with aConfigMap, which is basically a set of key-value pairs. This data can be made available so that a Pod
can read the data as environment variables or configuration data. AConfigMapis similar to aSecret, except they are not
base64 byte encoded arrays. They are stored as strings and can be read in serialized form.

### Create a ConfigMap

There are three different ways aConfigMapcan ingest data, from a literal value, from a file or from a directory of files.

1. We will create aConfigMapcontaining primary colors. We will create a series of files to ingest into theConfigMap.
    First, we create a directoryprimaryand populate it with four files. Then we create a file in our home directory with our
    favorite color.

#### 59


#### 60 CHAPTER 9. VOLUMES AND DATA

```
student@lfs458-node-1a0a:~$ mkdir primary
```
```
student@lfs458-node-1a0a:~$ echo c > primary/cyan
```
```
student@lfs458-node-1a0a:~$ echo m > primary/magenta
```
```
student@lfs458-node-1a0a:~$ echo y > primary/yellow
```
```
student@lfs458-node-1a0a:~$ echo k > primary/black
```
```
student@lfs458-node-1a0a:~$ echo "known as key" >> primary/black
```
```
student@lfs458-node-1a0a:~$ echo blue > favorite
```
2. Now we will create theConfigMapand populate it with the files we created as well as a literal value from the command
    line.

```
student@lfs458-node-1a0a:~$ kubectl create configmap colors \
--from-literal=text=black \
--from-file=./favorite \
--from-file=./primary/
configmap/colors created
```
3. View how the data is organized inside the cluster.

```
student@lfs458-node-1a0a:~$ kubectl get configmap colors
NAME DATA AGE
colors 6 30s
```
```
student@lfs458-node-1a0a:~$ kubectl get configmap colors -o yaml
apiVersion: v1
data:
black: |
k
known as key
cyan: |
c
favorite: |
blue
magenta: |
m
text: black
yellow: |
y
kind: ConfigMap
<output_omitted>
```
4. Now we can create a Pod to use theConfigMap. In this case a particular parameter is being defined as an environment
    variable.

```
student@lfs458-node-1a0a:~$ vim simpleshell.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
name: shell-demo
spec:
containers:
```
- name: nginx
    image: nginx
    env:


#### 9.1. LABS 61

- name: ilike
    valueFrom:
       configMapKeyRef:
          name: colors
          key: favorite
5. Create the Pod and view the environmental variable. After you view the parameter, exit out and delete the pod.

```
student@lfs458-node-1a0a:~$ kubectl create -f simpleshell.yaml
pod/shell-demo created
```
```
student@lfs458-node-1a0a:~$ kubectl exec -it shell-demo \
-- /bin/bash -c ’echo $ilike’
blue
```
```
student@lfs458-node-1a0a:~$ kubectl delete pod shell-demo
pod "shell-demo" deleted
```
6. All variables from a file can be included as environment variables as well. Comment out the previousenv:stanza
    and add a slightly differentenvFromto the file. Having new and old code at the same time can be helpful to see and
    understand the differences. Recreate the Pod, check all variables and delete the pod again. They can be found spread
    throughout the environment variable output.

```
student@lfs458-node-1a0a:~$ vim simpleshell.yaml
<output_omitted>
image: nginx
# env:
# - name: ilike
# valueFrom:
# configMapKeyRef:
# name: colors
# key: favorite
envFrom:
```
- configMapRef:
    name: colors

```
student@lfs458-node-1a0a:~$ kubectl create -f simpleshell.yaml
pod/shell-demo created
```
```
student@lfs458-node-1a0a:~$ kubectl exec -it shell-demo \
-- /bin/bash -c ’env’
HOSTNAME=shell-demo
NJS_VERSION=1.13.6.0.1.14-1~stretch
NGINX_VERSION=1.13.6-1~stretch
black=k
know as key
```
```
favorite=blue
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ kubectl delete pod shell-demo
pod "shell-demo" deleted
```
7. AConfigMapcan also be created from a YAML file. Create one with a few parameters to describe a car.

```
student@lfs458-node-1a0a:~$ vim car-map.yaml
```
```
apiVersion: v1
kind: ConfigMap
metadata:
name: fast-car
```

#### 62 CHAPTER 9. VOLUMES AND DATA

```
namespace: default
data:
car.make: Ford
car.model: Mustang
car.trim: Shelby
```
8. Create theConfigMapand verify the settings.

```
student@lfs458-node-1a0a:~$ kubectl create -f car-map.yaml
configmap/fast-car created
```
```
student@lfs458-node-1a0a:~$ kubectl get configmap fast-car -o yaml
apiVersion: v1
data:
car.make: Ford
car.model: Mustang
car.trim: Shelby
kind: ConfigMap
<output_omitted>
```
9. We will now make theConfigMapavailable to a Pod as a mounted volume. You can again comment out the previous
    environmental settings and add the following new stanza. Thecontainers:andvolumes:entries are indented the
    same number of spaces.

```
student@lfs458-node-1a0a:~$ vim simpleshell.yaml
<output_omitted>
spec:
containers:
```
- name: nginx
    image: nginx
    volumeMounts:
    - name: car-vol
       mountPath: /etc/cars
volumes:
- name: car-vol
configMap:
name: fast-car
<comment out rest of file>
10. Create the Pod again. Verify the volume exists and the contents of a file within. Due to the lack of a carriage return in
the file your next prompt may be on the same line as the output,Shelby.

```
student@lfs458-node-1a0a:~$ kubectl create -f simpleshell.yaml
pod "shell-demo" created
```
```
student@lfs458-node-1a0a:~$ kubectl exec -it shell-demo -- \
/bin/bash -c ’df -ha |grep car’
/dev/sda1 20G 4.7G 15G 25% /etc/cars
```
```
student@lfs458-node-1a0a:~$ kubectl exec -it shell-demo -- \
/bin/bash -c ’cat /etc/cars/car.trim’
Shelby
```
11. Delete thePodandConfigMapswe were using.

```
student@lfs458-node-1a0a:~$ kubectl delete pods shell-demo
pod "shell-demo" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl delete configmap fast-car colors
configmap "fast-car" deleted
configmap "colors" deleted
```

#### 9.1. LABS 63

### Exercise 9.2: Creating a Persistent NFS Volume (PV)

We will first deploy an NFS server. Once tested we will create a persistent NFS volume for containers to claim.

1. Install the software on your master node.

```
student@lfs458-node-1a0a:~$ sudo apt-get update && sudo \
apt-get install -y nfs-kernel-server
<output_omitted>
```
2. Make and populate a directory to be shared. Also give it similar permissions to/tmp/

```
student@lfs458-node-1a0a:~$ sudo mkdir /opt/sfw
```
```
student@lfs458-node-1a0a:~$ sudo chmod 1777 /opt/sfw/
```
```
student@lfs458-node-1a0a:~$ sudo bash -c \
’echo software > /opt/sfw/hello.txt’
```
3. Edit the NFS server file to share out the newly created directory. In this case we will share the directory with all. You can
    always **snoop** to see the inbound request in a later step and update the file to be more narrow.

```
student@lfs458-node-1a0a:~$ sudo vim /etc/exports
/opt/sfw/ *(rw,sync,no_root_squash,subtree_check)
```
4. Cause/etc/exportsto be re-read:

```
student@lfs458-node-1a0a:~$ sudo exportfs -ra
```
5. Test by mounting the resource from your **second** node.

```
student@lfs458-worker:~$ sudo apt-get -y install nfs-common
<output_omitted>
```
```
student@lfs458-worker:~$ showmount -e lfs458-node-1a0a
Export list for lfs458-node-1a0a:
/opt/sfw *
```
```
student@lfs458-worker:~$ sudo mount 10.128.0.3:/opt/sfw /mnt
```
```
student@lfs458-worker:~$ ls -l /mnt
total 4
-rw-r--r-- 1 root root 9 Sep 28 17:55 hello.txt
```
6. Return to the master node and create a YAML file for the object withkind,PersistentVolume. Use the hostname
    of the master server and the directory you created in the previous step. Only syntax is checked, an incorrect name
    or directory will not generate an error, but a Pod using the resource will not start. Note that theaccessModesdo not
    currently affect actual access and are typically used as labels instead.

```
student@lfs458-node-1a0a:~$ vim PVol.yaml
```
```
apiVersion: v1
kind: PersistentVolume
metadata:
name: pvvol-1
spec:
capacity:
storage: 1Gi
accessModes:
```
- ReadWriteMany
persistentVolumeReclaimPolicy: Retain


#### 64 CHAPTER 9. VOLUMES AND DATA

```
nfs:
path: /opt/sfw
server: lfs458-node-1a0a #<-- Edit to match master node
readOnly: false
```
7. Create the persistent volume, then verify its creation.

```
student@lfs458-node-1a0a:~$ kubectl create -f PVol.yaml
persistentvolume/pvvol-1 created
```
```
student@lfs458-node-1a0a:~$ kubectl get pv
NAME CAPACITY ACCESSMODES RECLAIMPOLICY STATUS
CLAIM STORAGECLASS REASON AGE
pvvol-1 1Gi RWX Retain Available 4s
```
### Exercise 9.3: Creating a Persistent Volume Claim (PVC)

Before Pods can take advantage of the new PV we need to create a **Persistent Volume Claim** ( **PVC** ).

1. Begin by determining if any currently exist.

```
student@lfs458-node-1a0a:~$ kubectl get pvc
No resources found.
```
2. Create a YAML file for the new pvc.

```
student@lfs458-node-1a0a:~$ vim pvc.yaml
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: pvc-one
spec:
accessModes:
```
- ReadWriteMany
resources:
    requests:
       storage: 200Mi
3. Create and verify the new pvc is bound. Note that the size is 1Gi, even though 200Mi was suggested. Only a volume of
at least that size could be used.

```
student@lfs458-node-1a0a:~$ kubectl create -f pvc.yaml
persistentvolumeclaim/pvc-one created
```
```
student@lfs458-node-1a0a:~$ kubectl get pvc
NAME STATUS VOLUME CAPACITY ACCESSMODES STORAGECLASS AGE
pvc-one Bound pvvol-1 1Gi RWX 4s
```
4. Look at the status of the pv again, to determine if it is in use. It should show a status of Bound.

```
student@lfs458-node-1a0a:~$ kubectl get pv
NAME CAPACITY ACCESSMODES RECLAIMPOLICY STATUS CLAIM STORAGECLASS REASON AGE
pvvol-1 1Gi RWX Retain Bound default/pvc-one 5m
```
5. Create a new deployment to use the pvc. We will copy and edit an existing deployment yaml file. We will change the
    deployment name then add a volumeMounts section under containers and volumes section to the general spec. The
    name used must match in both places, whatever name you use. The claimName must match an existing pvc. As shown
    in the following example.


#### 9.1. LABS 65

```
student@lfs458-node-1a0a:~$ cp first.yaml nfs-pod.yaml
```
```
student@lfs458-node-1a0a:~$ vim nfs-pod.yaml
```
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
annotations:
deployment.kubernetes.io/revision: "1"
generation: 1
labels:
run: nginx
name: nginx-nfs
namespace: default
resourceVersion: "1411"
spec:
replicas: 1
selector:
matchLabels:
run: nginx
strategy:
rollingUpdate:
maxSurge: 1
maxUnavailable: 1
type: RollingUpdate
template:
metadata:
creationTimestamp: null
labels:
run: nginx
spec:
containers:
```
- image: nginx
    imagePullPolicy: Always
    name: nginx
    volumeMounts:
    - name: nfs-vol
       mountPath: /opt
    ports:
    - containerPort: 80
       protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
volumes: #<<-- These four lines
- name: nfs-vol
    persistentVolumeClaim:
       claimName: pvc-one
dnsPolicy: ClusterFirst
restartPolicy: Always
schedulerName: default-scheduler
securityContext: {}
terminationGracePeriodSeconds: 30
6. Create the pod using the newly edited file.

```
student@lfs458-node-1a0a:~$ kubectl create -f nfs-pod.yaml
deployment.apps/nginx-nfs created
```
7. Look at the details of the pod. You may see thedaemonsetpods running as well.


#### 66 CHAPTER 9. VOLUMES AND DATA

```
student@lfs458-node-1a0a:~$ kubectl get pods
NAME READY STATUS RESTARTS AGE
nginx-nfs-1054709768-s8g28 1/1 Running 0 3m
```
```
student@lfs458-node-1a0a:~$ kubectl describe pod nginx-nfs-1054709768-s8g28
Name: nginx-nfs-1054709768-s8g28
Namespace: default
Node: lfs458-worker/10.128.0.5
```
```
<output_omitted>
```
```
Mounts:
/opt from nfs-vol (rw)
```
```
<output_omitted>
```
```
Volumes:
nfs-vol:
Type: PersistentVolumeClaim (a reference to a PersistentV...
ClaimName: pvc-one
ReadOnly: false
<output_omitted>
```
8. View the status of thePVC. It should show as bound.

```
student@lfs458-node-1a0a:~$ kubectl get pvc
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
pvc-one Bound pvvol-1 1Gi RWX 2m
```
### Exercise 9.4: Using a ResourceQuota to Limit PVC Count and Usage

The flexibility of cloud-based storage often requires limiting consumption among users. We will use theResourceQuotaobject
to both limit the total consumption as well as the number of persistent volume claims.

1. Begin by deleting the deployment we had created to use NFS, the pv and the pvc.

```
student@lfs458-node-1a0a:~$ kubectl delete deploy nginx-nfs
deployment.extensions "nginx-nfs" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl delete pvc pvc-one
persistentvolumeclaim "pvc-one" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl delete pv pvvol-1
persistentvolume "pvvol-1" deleted
```
2. Create a yaml file for theResourceQuotaobject. Set the storage limit to ten claims with a total usage of 500Mi.

```
student@lfs458-node-1a0a:~$ vim storage-quota.yaml
```
```
apiVersion: v1
kind: ResourceQuota
metadata:
name: storagequota
spec:
hard:
persistentvolumeclaims: "10"
requests.storage: "500Mi"
```

#### 9.1. LABS 67

3. Create a new namespace calledsmall. View the namespace information prior to the new quota. Either the long name
    with double dashes--namespaceor the nicknamenswork for the resource.

```
student@lfs458-node-1a0a:~$ kubectl create namespace small
namespace/small created
```
```
student@lfs458-node-1a0a:~$ kubectl describe ns small
Name: small
Labels: <none>
Annotations: <none>
Status: Active
```
```
No resource quota.
```
```
No resource limits.
```
4. Create a new pv and pvc in the small namespace.

```
student@lfs458-node-1a0a:~$ kubectl create -f PVol.yaml -n small
persistentvolume/pvvol-1 created
```
```
student@lfs458-node-1a0a:~$ kubectl create -f pvc.yaml -n small
persistentvolumeclaim/pvc-one created
```
5. Create the new resource quota, placing this object into thelow-usage-limitnamespace.

```
student@lfs458-node-1a0a:~$ kubectl create -f storage-quota.yaml \
-n small
resourcequota/storagequota created
```
6. Verify thesmallnamespace has quotas. Compare the output to the same command above.

```
student@lfs458-node-1a0a:~$ kubectl describe ns small
Name: small
Labels: <none>
Annotations: <none>
Status: Active
```
```
Resource Quotas
Name: storagequota
Resource Used Hard
-------- --- ---
persistentvolumeclaims 1 10
requests.storage 200Mi 500Mi
```
```
No resource limits.
```
7. Remove the namespace line from thenfs-pod.yamlfile. Should be around line 11 or so. This will allow us to pass
    other namespaces on the command line.

```
student@lfs458-node-1a0a:~$ vim nfs-pod.yaml
```
8. Create the container again.

```
student@lfs458-node-1a0a:~$ kubectl create -f nfs-pod.yaml \
-n small
deployment.apps/nginx-nfs created
```
9. Determine if the deployment has a running pod.


#### 68 CHAPTER 9. VOLUMES AND DATA

```
student@lfs458-node-1a0a:~$ kubectl get deploy --namespace=small
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
nginx-nfs 1 1 1 0 43s
```
```
student@lfs458-node-1a0a:~$ kubectl -n small describe deploy \
nginx-nfs
<output_omitted>
```
10. Look to see if the pods are ready.

```
student@lfs458-node-1a0a:~$ kubectl get po -n small
NAME READY STATUS RESTARTS AGE
nginx-nfs-2854978848-g3khf 1/1 Running 0 37s
```
11. Ensure the Pod is running and is using the NFS mounted volume. If you pass the namespace firstTabwill auto-complete
    the pod name.

```
student@lfs458-node-1a0a:~$ kubectl -n small describe po \
nginx-nfs-2854978848-g3khf
Name: nginx-nfs-2854978848-g3khf
Namespace: small
<output_omitted>
```
```
Mounts:
/opt from nfs-vol (rw)
<output_omitted>
```
12. View the quota usage of the namespace

```
student@lfs458-node-1a0a:~$ kubectl describe ns small
<output_omitted>
```
```
Resource Quotas
Name: storagequota
Resource Used Hard
-------- --- ---
persistentvolumeclaims 1 10
requests.storage 200Mi 500Mi
```
```
No resource limits.
```
13. Create a 300M file inside of the/opt/sfwdirectory on the host and view the quota usage again. Note that with NFS the
    size of the share is not counted against the deployment.

```
student@lfs458-node-1a0a:~$ sudo dd if=/dev/zero \
of=/opt/sfw/bigfile bs=1M count=300
300+0 records in
300+0 records out
314572800 bytes (315 MB, 300 MiB) copied, 0.196794 s, 1.6 GB/s
```
```
student@lfs458-node-1a0a:~$ kubectl describe ns small
<output_omitted>
Resource Quotas
Name: storagequota
Resource Used Hard
-------- --- ---
persistentvolumeclaims 1 10
requests.storage 200Mi 500Mi
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ du -h /opt/
301M /opt/sfw
```

#### 9.1. LABS 69

```
41M /opt/cni/bin
41M /opt/cni
341M /opt/
```
14. Now let us illustrate what happens when a deployment requests more than the quota. Begin by shutting down the
    existing deployment.

```
student@lfs458-node-1a0a:~$ kubectl -n small get deploy
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
nginx-nfs 1 1 1 1 11m
```
```
student@lfs458-node-1a0a:~$ kubectl -n small delete deploy nginx-nfs
deployment.extensions "nginx-nfs" deleted
```
15. Once the Pod has shut down view the resource usage of the namespace again. Note the storage did not get cleaned
    up when the pod was shut down.

```
student@lfs458-node-1a0a:~$ kubectl describe ns small
<output_omitted>
Resource Quotas
Name: storagequota
Resource Used Hard
-------- --- ---
persistentvolumeclaims 1 10
requests.storage 200Mi 500Mi
```
16. Remove the pvc then view the pv it was using. Note theRECLAIM POLICYandSTATUS.

```
student@lfs458-node-1a0a:~$ kubectl get pvc -n small
NAME STATUS VOLUME CAPACITY ACCESSMODES STORAGECLASS AGE
pvc-one Bound pvvol-1 1Gi RWX 19m
```
```
student@lfs458-node-1a0a:~$ kubectl -n small delete pvc pvc-one
persistentvolumeclaim "pvc-one" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl -n small get pv
NAME CAPACITY ACCESSMODES RECLAIMPOLICY STATUS CLAIM
STORAGECLASS REASON AGE
pvvol-1 1Gi RWX Retain Released small/pvc-one 44m
```
17. Dynamically provisioned storage uses theReclaimPolicyof theStorageClasswhich could beDelete,Retain, or
    some types allowRecycle. Manually created persistent volumes default toRetainunless set otherwise at creation.
    The default storage policy is to retain the storage to allow recovery of any data. To change this begin by viewing the
    yaml output.

```
student@lfs458-node-1a0a:~$ kubectl get pv/pvvol-1 -o yaml
....
path: /opt/sfw
server: lfs458-node-1a0a
persistentVolumeReclaimPolicy: Retain
status:
phase: Released
```
18. Currently we will need to delete and re-create the object. Future development on a deleter plugin is planned. We will
    re-create the volume and allow it to use theRetainpolicy, then change it once running.

```
student@lfs458-node-1a0a:~$ kubectl delete pv/pvvol-1
persistentvolume "pvvol-1" deleted
```
```
student@lfs458-node-1a0a:~$ grep Retain PVol.yaml
persistentVolumeReclaimPolicy: Retain
```
```
student@lfs458-node-1a0a:~$ kubectl create -f PVol.yaml
persistentvolume "pvvol-1" created
```

#### 70 CHAPTER 9. VOLUMES AND DATA

19. We will usekubectl patchto change the retention policy toDelete. The yaml output from before can be helpful in
    getting the correct syntax.

```
student@lfs458-node-1a0a:~$ kubectl patch pv pvvol-1 -p \
’{"spec":{"persistentVolumeReclaimPolicy":"Delete"}}’
persistentvolume/pvvol-1 patched
```
```
student@lfs458-node-1a0a:~$ kubectl get pv/pvvol-1
NAME CAPACITY ACCESSMODES RECLAIMPOLICY STATUS CLAIM
STORAGECLASS REASON AGE
pvvol-1 1Gi RWX Delete Available 2m
```
20. View the current quota settings.

```
student@lfs458-node-1a0a:~$ kubectl describe ns small
.
requests.storage 0 500Mi
```
21. Create the pvc again. Even with no pods running, note the resource usage.

```
student@lfs458-node-1a0a:~$ kubectl -n small create -f pvc.yaml
persistentvolumeclaim/pvc-one created
```
```
student@lfs458-node-1a0a:~$ kubectl describe ns small
.
requests.storage 200Mi 500Mi
```
22. Remove the existing quota from the namespace.

```
student@lfs458-node-1a0a:~$ kubectl -n small get resourcequota
NAME CREATED AT
storagequota 2018-08-01T04:10:02Z
```
```
student@lfs458-node-1a0a:~$ kubectl -n small delete \
resourcequota storagequota
resourcequota "storagequota" deleted
```
23. Edit the storagequota.yaml file and lower the capacity to 100Mi.

```
student@lfs458-node-1a0a:~$ vim storage-quota.yaml
```
```
.
requests.storage: "100Mi"
```
24. Create and verify the new storage quota. Note the hard limit has already been exceeded.

```
student@lfs458-node-1a0a:~$ kubectl create -f storage-quota.yaml -n small
resourcequota/storagequota created
```
```
student@lfs458-node-1a0a:~$ kubectl describe ns small
.
persistentvolumeclaims 1 10
requests.storage 200Mi 100Mi
```
```
No resource limits.
```
25. Create the deployment again. View the deployment. Note there are no errors seen.


#### 9.1. LABS 71

```
student@lfs458-node-1a0a:~$ kubectl create -f nfs-pod.yaml \
-n small
deployment.apps/nginx-nfs created
```
```
student@lfs458-node-1a0a:~$ kubectl -n small describe deploy/nginx-nfs
Name: nginx-nfs
Namespace: small
<output_omitted>
```
26. Examine the pods to see if they are actually running.

```
student@lfs458-node-1a0a:~$ kubectl -n small get po
NAME READY STATUS RESTARTS AGE
nginx-nfs-2854978848-vb6bh 1/1 Running 0 58s
```
27. As we were able to deploy more pods even with apparent hard quota set, let us test to see if the reclaim of storage takes
    place. Remove the deployment and the persistent volume claim.

```
student@lfs458-node-1a0a:~$ kubectl -n small delete deploy nginx-nfs
deployment.extensions "nginx-nfs" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl -n small delete pvc/pvc-one
persistentvolumeclaim "pvc-one" deleted
```
28. View if the persistent volume exists. You will see it attempted a removal, but failed. If you look closer you will find the
    error has to do with the lack of adeleter volume pluginfor NFS. Other storage protocols have a plugin.

```
student@lfs458-node-1a0a:~$ kubectl -n small get pv
NAME CAPACITY ACCESSMODES RECLAIMPOLICY STATUS CLAIM
STORAGECLASS REASON AGE
pvvol-1 1Gi RWX Delete Failed small/pvc-one 20m
```
29. Ensure the deployment, pvc and pv are all removed.

```
student@lfs458-node-1a0a:~$ kubectl delete pv/pvvol-1
persistentvolume "pvvol-1" deleted
```
30. Edit the persistent volume YAML file and change thepersistentVolumeReclaimPolicy:to Recycle.

```
student@lfs458-node-1a0a:~$ vim PVol.yaml
....
persistentVolumeReclaimPolicy: Recycle
....
```
31. Add aLimitRangeto the namespace and attempt to create the persistent volume and persistent volume claim again.
    We can use theLimitRangewe used earlier.

```
student@lfs458-node-1a0a:~$ kubectl -n small create -f \
low-resource-range.yaml
limitrange/low-resource-range created
```
32. View the settings for the namespace. Both quotas and resource limits should be seen.

```
student@lfs458-node-1a0a:~$ kubectl describe ns small
<output_omitted>
Resource Limits
Type Resource Min Max Default Request Default Limit ...
---- -------- --- --- --------------- ------------- -...
Container cpu - - 500m 1 -
Container memory - - 100Mi 500Mi -
```

#### 72 CHAPTER 9. VOLUMES AND DATA

33. Create the persistent volume again. View the resource. Note theReclaim PolicyisRecycle.

```
student@lfs458-node-1a0a:~$ kubectl -n small create -f PVol.yaml
persistentvolume/pvvol-1 created
```
```
student@lfs458-node-1a0a:~$ kubectl get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS ...
pvvol-1 1Gi RWX Recycle Available ...
```
34. Attempt to create the persistent volume claim again. The quota only takes effect if there is also a resource limit in effect.

```
student@lfs458-node-1a0a:~$ kubectl -n small create -f pvc.yaml
Error from server (Forbidden): error when creating "pvc.yaml":
persistentvolumeclaims "pvc-one" is forbidden: exceeded quota:
storagequota, requested: requests.storage=200Mi, used:
requests.storage=0, limited: requests.storage=100Mi
```
35. Edit theresourcequotato increase therequests.storageto 500mi.

```
student@lfs458-node-1a0a:~$ kubectl -n small edit resourcequota
....
spec:
hard:
persistentvolumeclaims: "10"
requests.storage: 500Mi
status:
hard:
persistentvolumeclaims: "10"
....
```
36. Create thepvcagain. It should work this time. Then create thedeploymentagain.

```
student@lfs458-node-1a0a:~$ kubectl -n small create -f pvc.yaml
persistentvolumeclaim/pvc-one created
```
```
student@lfs458-node-1a0a:~$ kubectl -n small create -f nfs-pod.yaml
deployment.apps/nginx-nfs created
```
37. View the namespace settings.

```
student@lfs458-node-1a0a:~$ kubectl describe ns small
<output_omitted>
```
38. Delete thedeployment. View the status of thepvandpvc.

```
student@lfs458-node-1a0a:~$ kubectl -n small delete deploy nginx-nfs
deployment.extensions "nginx-nfs" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl get pvc -n small
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE
pvc-one Bound pvvol-1 1Gi RWX 7m
```
```
student@lfs458-node-1a0a:~$ kubectl -n small get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORA...
pvvol-1 1Gi RWX Recycle Bound small/pvc-one ...
```
39. Delete thepvcand check the status of thepv. It should show asAvailable.

```
student@lfs458-node-1a0a:~$ kubectl -n small delete pvc pvc-one
persistentvolumeclaim "pvc-one" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl -n small get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORA...
pvvol-1 1Gi RWX Recycle Available ...
```

#### 9.1. LABS 73

40. Remove thepvand any other resources created during this lab.

```
student@lfs458-node-1a0a:~$ kubectl delete pv pvvol-1
persistentvolume "pvvol-1" deleted
```

#### 74 CHAPTER 9. VOLUMES AND DATA


**Chapter 10**

**Ingress**

### 10.1 Labs

### Exercise 10.1: Advanced Service Exposure

### Configure an Ingress Controller

With such a fast changing project, it is important to keep track of updates. The main place to find documentation of the current
version ishttps://kubernetes.io/.

1. If you have a large number of services to expose outside of the cluster, or to expose a low-number port on the host node
    you can deploy an ingress controller or a service mesh. While **nginx** and **GCE** have controllers officially supported by
    Kubernetes.io, the **Traefik** ingress controller is easier to install. At the moment.

```
student@lfs458-node-1a0a:~$ kubectl create deployment secondapp \
--image=nginx
```
2. Find the labels currently in use by the deployment. We will use them to tie traffic from the ingress controller to the proper
    service.

```
student@lfs458-node-1a0a:~$ kubectl get deployments secondapp -o yaml |grep label -A2
labels:
app: secondapp
name: secondapp
--
labels:
app: secondapp
spec:
```
3. Expose the new server as a NodePort.

```
student@lfs458-node-1a0a:~$ kubectl expose deployment secondapp \
--type=NodePort --port=80
```
#### 75


#### 76 CHAPTER 10. INGRESS

4. As we have RBAC configured we need to make sure the controller will run and be able to work with all necessary ports,
    endpoints and resources. Create a YAML file to declare aclusterroleand aclusterrolebinding.

```
student@lfs458-node-1a0a:~$ vim ingress.rbac.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
name: traefik-ingress-controller
rules:
```
- apiGroups:
    - ""
resources:
- services
- endpoints
- secrets
verbs:
- get
- list
- watch
- apiGroups:
    - extensions
resources:
- ingresses
verbs:
- get
- list
- watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
name: traefik-ingress-controller
roleRef:
apiGroup: rbac.authorization.k8s.io
kind: ClusterRole
name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
name: traefik-ingress-controller
namespace: kube-system
5. Create the new role and binding.

```
student@lfs458-node-1a0a:~$ kubectl create -f ingress.rbac.yaml
clusterrole.rbac.authorization.k8s.io "traefik-ingress-controller" created
clusterrolebinding.rbac.authorization.k8s.io "traefik-ingress-controller" created
```
6. Create the Traefik controller. We will use a script directly from their website. This URL has a shorter version below:
    https://raw.githubusercontent.com/containous/traefik/master/\examples/k8s/traefik-ds.yaml

```
student@lfs458-node-1a0a:~$ wget https://tinyurl.com/yawpexdt -O traefik-ds.yaml
```
7. We need to take out some security context settings, such that the diff output between the new and old would be true.
    Add thehostNetworkline and remove thesecurityContextlines. The indentation forhostNetworkshould line up
    with thecontainers:line.

```
student@lfs458-node-1a0a:~$ vim traefik-ds.yaml
diff traefik-ds.yaml.1 ds/traefik-ds.yaml
23a24 ## Add this line
> hostNetwork: true
34,39d34 ## Remove these lines
< securityContext:
< capabilities:
```

#### 10.1. LABS 77

```
< drop:
< - ALL
< add:
< - NET_BIND_SERVICE
```
8. Then create the ingress controller using **kubectl create**

```
student@lfs458-node-1a0a:~$ kubectl create -f traefik-ds.yaml
serviceaccount "traefik-ingress-controller" created
daemonset.extensions "traefik-ingress-controller" created
service "traefik-ingress-service" created
```
9. Now that there is a new controller we need to pass some rules, so it knows how to handle requests. Note that the host
    mentioned is [http://www.example.com,](http://www.example.com,) which is probably not your node name. We will pass a false header when testing. Also
    the service name needs to match thesecondapplabel we found in an earlier step.

```
student@lfs458-node-1a0a:~$ vim ingress.rule.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
name: ingress-test
annotations:
kubernetes.io/ingress.class: traefik
spec:
rules:
```
- host: [http://www.example.com](http://www.example.com)
    [http:](http:)
       paths:
       - backend:
          serviceName: secondapp
          servicePort: 80
path: /
10. Now ingest the rule into the cluster.

```
student@lfs458-node-1a0a:~$ kubectl create -f ingress.rule.yaml
ingress.extensions "ingress-test" created
```
11. We should be able to test the internal and external IP addresses, and see the nginx welcome page. The loadbalancer
    would present the traffic, a **curl** request in this case, to the externally facing interface. Use **ip a** to find the IP address
    of the interface which would face the loadbalancer. In this example the interface would beens4, and the IP would be
    10.128.0.7.

```
student@lfs458-node-1a0a:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid_lft forever preferred_lft forever
inet6 ::1/128 scope host
valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
link/ether 42:01:0a:80:00:03 brd ff:ff:ff:ff:ff:ff
inet 10.128.0.7/32 brd 10.128.0.3 scope global ens4
valid_lft forever preferred_lft forever
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ curl -H "Host: http://www.example.com" http://10.128.0.7/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
```

#### 78 CHAPTER 10. INGRESS

```
student@lfs458-node-1a0a:~$ curl -H "Host: http://www.example.com" http://35.193.3.179
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
<output_omitted>
```
12. At this point we would keep adding more and more web servers. Well configure one more, which would then be a
    process continued as many times as desired.
    Begin by deploying another nginx server. Give it a label and expose port 80.

```
student@lfs458-node-1a0a:~$ kubectl create deployment thirdpage --image=nginx
deployment.apps/thirdpage created
```
13. Find the label for the new deployment. Look for thename:, which would bethirdpagein this example.

```
student@lfs458-node-1a0a:~$ kubectl get deployment thirdpage -o yaml |grep -A2 Label
labels:
app: thirdpage
name: thirdpage
--
labels:
app: thirdpage
spec:
```
14. Expose the new server as aNodePort.

```
student@lfs458-node-1a0a:~$ kubectl expose deployment \
thirdpage --type=NodePort --port=80
service/thirdpage exposed
```
15. Now we will customize the installation. Run a bash shell inside the new pod. Your pod name will end differently. Install
    **vim** inside the container then edit theindex.htmlfile of nginx so that the title of the web page will beThird Page.

```
student@lfs458-node-1a0a:~$ kubectl exec -it thirdpage-5cf8d67664-zcmfh -- /bin/bash
```
```
root@thirdpage-5cf8d67664-zcmfh:/# apt-get update
<output_omitted>
```
```
root@thirdpage-5cf8d67664-zcmfh:/# apt-get install vim -y
<output_omitted>
```
```
root@thirdpage-5cf8d67664-zcmfh:/# vim /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>Third Page</title>
<style>
<output_omitted>
```
16. Edit the ingress rules to point thethridpageservice. Use theserviceNamewe found in an earlier step ofthirdpage.

```
student@lfs458-node-1a0a:~$ kubectl edit ingress ingress-test
<output_omitted>
```
- host: [http://www.example.com](http://www.example.com)
    [http:](http:)
       paths:
       - backend:
          serviceName: secondapp
          servicePort: 80
path: /


#### 10.1. LABS 79

- host: thirdpage.org
    [http:](http:)
       paths:
       - backend:
          serviceName: thirdpage
          servicePort: 80
path: /
status:
<output_omitted>
17. Test the second hostname using **curl** locally as well as from a remote system.

```
student@lfs458-node-1a0a:~$ curl -H "Host: thirdpage.org" http://10.128.0.7/
<!DOCTYPE html>
<html>
<head>
<title>Third Page</title>
<style>
<output_omitted>
```

#### 80 CHAPTER 10. INGRESS


**Chapter 11**

**Scheduling**

### 11.1 Labs

### Exercise 11.1: Assign Pods Using Labels

### Overview

While allowing the system to distribute Pods on your behalf is typically the best route, you may want to determine which nodes
a Pod will use. For example you may have particular hardware requirements to meet for the workload. You may want to assign
VIP Pods to new, faster hardware and everyone else to older hardware.

In this exercise we will uselabelsto schedule Pods to a particular node. Then we will exploretaintsto have more flexible
deployment in a large environment.

### Assign Pods Using Labels

1. Begin by getting a list of the nodes. They should be in the ready state and without added labels or taints.

```
student@lfs458-node-1a0a:~$ kubectl get nodes
NAME STATUS ROLES AGE VERSION
lfs458-node-1a0a Ready master 2d v1.12.1
lfs458-worker Ready <none> 2d v1.12.1
```
2. View the current labels and taints for the nodes.

```
student@lfs458-node-1a0a:~$ kubectl describe nodes |grep -i label
Labels: beta.kubernetes.io/arch=amd64
Labels: beta.kubernetes.io/arch=amd64
```
```
student@lfs458-node-1a0a:~$ kubectl describe nodes |grep -i taint
Taints: <none>
Taints: <none>
```
#### 81


#### 82 CHAPTER 11. SCHEDULING

3. Verify there are no deployments running, outside of thekube-systemnamespace. If there are, delete them. Then get
    a count of how many containers are running on both the master and secondary nodes. There are about 24 containers
    running on the master in the following example, and eight running on the worker. There are status lines which increase
    the **wc** count. You may have more or less, depending on previous labs and cleaning up of resources.

```
student@lfs458-node-1a0a:~$ kubectl get deployments --all-namespaces
NAMESPACE NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
default secondapp 1 1 1 1 37m
default thirdpage 1 1 1 1 14m
kube-system calico-typha 0 0 0 0 2d15h
kube-system coredns 2 2 2 2 2d15h
low-usage-limit limited-hog 1 1 1 1 1d29m
```
```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
24
```
```
student@lfs458-worker:~$ sudo docker ps |wc -l
14
```
4. For the purpose of the exercise we will assign the master node to be VIP hardware and the secondary node to be for
    others.

```
student@lfs458-node-1a0a:~$ kubectl label nodes lfs458-node-1a0a status=vip
node/lfs458-node-1a0a labeled
```
```
student@lfs458-node-1a0a:~$ kubectl label nodes lfs458-worker status=other
node/lfs458-worker labeled
```
5. Verify your settings. You will also find there are some built in labels such as hostname, os and architecture type. The
    output below appears on multiple lines for readability.

```
student@lfs458-node-1a0a:~$ kubectl get nodes --show-labels
NAME STATUS ROLES AGE VERSION LABELS
lfs458-node-1a0a Ready master 2d v1.12.1 beta.kubernetes.io/arch=
amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=lfs458-node-1a0a,
node-role.kubernetes.io/master=,status=vip
lfs458-worker Ready <none> 2d v1.12.1 beta.kubernetes.io/arch=
amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=lfs458-worker,status=other
```
6. Createvip.yamlto spawn fourbusyboxcontainers which sleep the whole time. Include thenodeSelectorentry.

```
student@lfs458-node-1a0a:~$ vim vip.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
name: vip
spec:
containers:
```
- name: vip1
    image: busybox
    args:
    - sleep
    - "1000000"
- name: vip2
    image: busybox
    args:
    - sleep
    - "1000000"
- name: vip3


#### 11.1. LABS 83

```
image: busybox
args:
```
- sleep
- "1000000"
- name: vip4
image: busybox
args:
- sleep
- "1000000"
nodeSelector:
status: vip
7. Deploy the new pod. Verify the containers have been created on the master node. It may take a few seconds for all the
containers to spawn. Check both the master and the secondary nodes.

```
student@lfs458-node-1a0a:~$ kubectl create -f vip.yaml
pod/vip created
```
```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
29
```
```
student@lfs458-worker:~$ sudo docker ps |wc -l
8
```
8. Delete the pod then edit the file, commenting out thenodeSelectorlines. It may take a while for the containers to fully
    terminate.

```
student@lfs458-node-1a0a:~$ kubectl delete pod vip
pod "vip" deleted
```
```
student@lfs458-node-1a0a:~$ vim vip.yaml
....
# nodeSelector:
# status: vip
```
9. Create the pod again. Containers should now be spawning on both nodes. You may see pods for thedaemonsetsas
    well.

```
student@lfs458-node-1a0a:~$ kubectl get pods
NAME READY STATUS RESTARTS AGE
ds-one-bdqst 1/1 Running 0 145m
ds-one-t2t7z 1/1 Running 0 158m
secondapp-85765cd95c-2q9sx 1/1 Running 0 43m
thirdpage-7c9b56bfdd-2q5pr 1/1 Running 0 20m
```
```
student@lfs458-node-1a0a:~$ kubectl create -f vip.yaml
pod/vip created
```
10. Determine where the new containers have been deployed. They should be more evenly spread this time.

```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
24
```
```
student@lfs458-worker:~$ sudo docker ps |wc -l
19
```
11. Create another file for other users. Change the names from vip to others, and uncomment the nodeSelector lines.

```
student@lfs458-node-1a0a:~$ cp vip.yaml other.yaml
```
```
student@lfs458-node-1a0a:~$ sed -i s/vip/other/g other.yaml
```

#### 84 CHAPTER 11. SCHEDULING

```
student@lfs458-node-1a0a:~$ vim other.yaml
.
nodeSelector:
status: other
```
12. Create the other containers. Determine where they deploy.

```
student@lfs458-node-1a0a:~$ kubectl create -f other.yaml
pod/other created
```
```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
24
```
```
student@lfs458-worker:~$ sudo docker ps |wc -l
24
```
13. Shut down both pods and verify they terminated. Only our previous pods should be found.

```
student@lfs458-node-1a0a:~$ kubectl delete pods vip other
pod "vip" deleted
pod "other" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl get pods
NAME READY STATUS RESTARTS AGE
ds-one-bdqst 1/1 Running 0 153m
ds-one-t2t7z 1/1 Running 0 166m
secondapp-85765cd95c-2q9sx 1/1 Running 0 51m
thirdpage-7c9b56bfdd-2q5pr 1/1 Running 0 28m
```
### Exercise 11.2: Using Taints to Control Pod Deployment

Use taints to manage where Pods are deployed or allowed to run. In addition to assigning a Pod to a group of nodes, you may
also want to limit usage on a node or fully evacuate Pods. Using taints is one way to achieve this. You may remember that the
master node begins with aNoScheduletaint. We will work with three taints to limit or remove running pods.

1. Verify that the master and secondary node have the minimal number of containers running.

```
student@lfs458-node-1a0a:~$ kubectl delete deployment secondapp \
thirdpage
deployment.extensions "secondapp" deleted
deployment.extensions "thirdpage" deleted
```
2. Create a deployment which will deploy eight **nginx** containers. Begin by creating aYAMLfile.

```
student@lfs458-node-1a0a:~$ vim taint.yaml
```
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
name: taint-deployment
spec:
replicas: 8
template:
metadata:
labels:
app: nginx
spec:
containers:
```
- name: nginx


#### 11.1. LABS 85

```
image: nginx:1.7.9
ports:
```
- containerPort: 80
3. Apply the file to create the deployment.

```
student@lfs458-node-1a0a:~$ kubectl apply -f taint.yaml
deployment.apps/taint-deployment created
```
4. Determine where the containers are running. In the following example three have been deployed on the master node
    and five on the secondary node. Remember there will be other housekeeping containers created as well. Your numbers
    may be slightly different.

```
student@lfs458-node-1a0a:~$ sudo docker ps |grep nginx
00c1be5df1e7 nginx@sha256:e3456c851a152494c3e..
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
28
student@lfs458-worker:~$ sudo docker ps |wc -l
26
```
5. Delete the deployment. Verify the containers are gone.

```
student@lfs458-node-1a0a:~$ kubectl delete deployment taint-deployment
deployment.extensions "taint-deployment" deleted
```
```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
24
```
6. Now we will use a taint to affect the deployment of new containers. There are three taints, NoSchedule,
    PreferNoScheduleandNoExecute. The taints having to do with schedules will be used to determine newly deployed
    containers, but will not affect running containers. The use ofNoExecutewill cause running containers to move.
    Taint the secondary node, verify it has the taint then create the deployment again. We will use the key ofbubbato
    illustrate the key name is just some string an admin can use to track Pods.

```
student@lfs458-node-1a0a:~$ kubectl taint nodes lfs458-worker \
bubba=value:PreferNoSchedule
node/lfs458-worker tainted
```
```
student@lfs458-node-1a0a:~$ kubectl describe node |grep Taint
Taints: bubba=value:PreferNoSchedule
Taints: <none>
```
```
student@lfs458-node-1a0a:~$ kubectl apply -f taint.yaml
deployment.apps/taint-deployment created
```
7. Locate where the containers are running. We can see that more containers are on the master, but there still were some
    created on the secondary. Delete the deployment when you have gathered the numbers.

```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
32
```
```
student@lfs458-worker:~$ sudo docker ps |wc -l
22
```
```
student@lfs458-node-1a0a:~$ kubectl delete deployment taint-deployment
deployment.extensions "taint-deployment" deleted
```
8. Remove the taint, verify it has been removed. Note that the key is used with a minus sign appended to the end.


#### 86 CHAPTER 11. SCHEDULING

```
student@lfs458-node-1a0a:~$ kubectl taint nodes lfs458-worker bubba-
node/lfs458-worker untainted
```
```
student@lfs458-node-1a0a:~$ kubectl describe node |grep Taint
Taints: <none>
Taints: <none>
```
9. This time use theNoScheduletaint, then create the deployment again. The secondary node should not have any new
    containers, with only daemonsets and other essential pods running.

```
student@lfs458-node-1a0a:~$ kubectl taint nodes lfs458-worker \
bubba=value:NoSchedule
node/lfs458-worker tainted
```
```
student@lfs458-node-1a0a:~$ kubectl apply -f taint.yaml
deployment.apps/taint-deployment created
```
```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
24
```
```
student@lfs458-worker:~$ sudo docker ps |wc -l
14
```
10. Remove the taint and delete the deployment. When you have determined that all the containers are terminated create
    the deployment again. Without any taint the containers should be spread across both nodes.

```
student@lfs458-node-1a0a:~$ kubectl delete deployment taint-deployment
deployment.extensions "taint-deployment" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl taint nodes lfs458-worker bubba-
node/lfs458-worker untainted
```
```
student@lfs458-node-1a0a:~$ kubectl apply -f taint.yaml
deployment.apps/taint-deployment created
```
```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
32
```
```
student@lfs458-worker:~$ sudo docker ps |wc -l
22
```
11. Now use theNoExecuteto taint the secondary node. Wait a minute then determine if the containers have moved.
    The DNS containers can take a while to shutdown. A few containers will remain on the worker node to continue
    communication from the cluster.

```
student@lfs458-node-1a0a:~$ kubectl taint nodes lfs458-worker \
bubba=value:NoExecute
node "lfs458-worker" tainted
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
32
```
```
student@lfs458-worker:~$ sudo docker ps |wc -l
6
```
12. Remove the taint. Wait a minute. Note that all of the containers did not return to their previous placement.

```
student@lfs458-node-1a0a:~$ kubectl taint nodes lfs458-worker bubba-
node/lfs458-worker untainted
```
```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
32
```

#### 11.1. LABS 87

```
student@lfs458-worker:~$ sudo docker ps |wc -l
6
```
13. In addition to the ability to taint a node you can also set the status to drain. First view the status, then destroy the existing
    deployment. Note that the status reports Ready, even though it will not allow containers to be executed. Also note that
    the output mentioned thatDaemonSet-managed pods are not affected by default, as we saw in an earlier lab. This time
    lets take a closer look at what happens to existing pods and nodes.
    Existing containers are not moved, but no new containers are created. You may receive an error
    error: unable to drain node "<your node>", aborting command...

```
student@lfs458-node-1a0a:~$ kubectl get nodes
NAME STATUS ROLES AGE VERSION
lfs458-node-1a0a Ready master 2d v1.12.1
lfs458-worker Ready <none> 2d v1.12.1
```
```
student@lfs458-node-1a0a:~$ kubectl drain lfs458-worker
node/lfs458-worker cordoned
error: DaemonSet-managed pods (use --ignore-daemonsets to ignore): kube-flannel-ds-fx3tx, kube-proxy-q2q4k
```
14. Verify the state change of the node. It should indicate no new Pods will be scheduled.

```
student@lfs458-node-1a0a:~$ kubectl get nodes
NAME STATUS ROLES AGE VERSION
lfs458-node-1a0a Ready master 2d v1.12.1
lfs458-worker Ready,SchedulingDisabled <none> 2d v1.12.1
```
15. Delete the deployment to destroy the current Pods.

```
student@lfs458-node-1a0a:~$ kubectl delete deployment taint-deployment
deployment.extensions "taint-deployment" deleted
```
16. Create the deployment again and determine where the containers have been deployed.

```
student@lfs458-node-1a0a:~$ kubectl apply -f taint.yaml
deployment.apps/taint-deployment created
```
```
student@lfs458-node-1a0a:~$ sudo docker ps |wc -l
44
```
17. Return the status to Ready, then destroy and create the deployment again. The containers should be spread across the
    nodes. Begin by removing the cordon on the node.

```
student@lfs458-node-1a0a:~$ kubectl uncordon lfs458-worker
node/lfs458-worker uncordoned
```
```
student@lfs458-node-1a0a:~$ kubectl get nodes
NAME STATUS ROLES AGE VERSION
lfs458-node-1a0a Ready master 2d v1.12.1
lfs458-worker Ready <none> 2d v1.12.1
```
18. Delete and re-create the deployment.

```
student@lfs458-node-1a0a:~$ kubectl delete deployment taint-deployment
deployment.extensions "taint-deployment" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl apply -f taint.yaml
deployment.apps/taint-deployment created
```

#### 88 CHAPTER 11. SCHEDULING

19. View the **docker ps** output again. Both nodes should have almost the same number of containers deployed. The master
    will have a few more, due to its role.
20. Remove the deployment a final time to free up resources.

```
student@lfs458-node-1a0a:~$ kubectl delete deployment taint-deployment
deployment.extensions "taint-deployment" deleted
```

**Chapter 12**

**Logging and Troubleshooting**

### 12.1 Labs

### Exercise 12.1: Review Log File Locations

### Overview

In addition to various logs files and command output, you can use **journalctl** to view logs from the node perspective. We will
view common locations of log files, then a command to view container logs. There are other logging options, such as the use
of a **sidecar** container dedicated to loading the logs of another container in a pod.

Whole cluster logging is not yet available with Kubernetes. Outside software is typically used, such as **Fluentd** , part of
https://fluentd.org/, which is another member project of **CNCF** , like Kubernetes.

### Review Log File Locations

Take a quick look at the following log files and web sites. As server processes move from node level to running in containers
the logging also moves.

1. If using a **systemd** based Kubernetes cluster view the node level logs for **kubelet** , the local Kubernetes agent. Each
    node will have different contents as this is node specific.

```
student@lfs458-node-1a0a:~$ journalctl -u kubelet |less
<output_omitted>
```
2. Major Kubernetes processes now run in containers. You can view them from the container or the pod perspective.
    Use the **find** command to locate the **kube-apiserver** log. Your output will be different, but will be very long. Once
    you locate the files use the **diff** utility to compare them. There should be no difference, as they are symbolic links to
    /var/log/pods/. If you follow the links the log files are unique.

#### 89


#### 90 CHAPTER 12. LOGGING AND TROUBLESHOOTING

```
student@lfs458-node-1a0a:~$ sudo find / -name "*apiserver*log"
/var/log/containers/kube-apiserver-u16-12-1-dcb8_kube-system_kube-apiserver-
eddae7079382cd382cd55f8f46b192565dd16b6858206039d49b1ad4693c2a10.log
/var/log/containers/kube-apiserver-u16-12-1-dcb8_kube-system_kube-apiserver-
d00a48877af4ed4c7f8eedf2c7805c77cfabb31fcb453f7d89ffa52fc6ea5f36.log
```
```
student@lfs458-node-1a0a:~$ sudo diff /var/log/containers/kube-apiserver-u16-
12-1-dcb8_kube-system_kube-apiserver-eddae7079382cd382cd55f8f46b192565dd16b68
58206039d49b1ad4693c2a10.log /var/log/containers/kube-apiserver-u16-12-1-
dcb8_kube-system_kube-apiserver-d00a48877af4ed4c7f8eedf2c7805c77cfabb31fcb453
f7d89ffa52fc6ea5f36.log
<output_omitted>
```
3. Take a look at the log file.

```
student@lfs458-node-1a0a:~$ sudo less /var/log/containers/kube-apiserver-u16-
12-1-dcb8_kube-system_kube-apiserver-d00a48877af4ed4c7f8eedf2c7805c77cfabb31f
cb453f7d89ffa52fc6ea5f36.log
```
4. Search for and review other log files forkube-dns,kube-flannel, andkube-proxy.
5. If not on a Kubernetes cluster using **systemd** you can view the text files on the master node.

```
(a)/var/log/kube-apiserver.log
Responsible for serving the API
(b)/var/log/kube-scheduler.log
Responsible for making scheduling decisions
(c)/var/log/kube-controller-manager.log
Controller that manages replication controllers
```
```
6./var/log/containers
Various container logs
```
```
7./var/log/pods/
More log files for current Pods.
```
8. Worker Nodes Files (on non- **systemd** systems)

```
(a)/var/log/kubelet.log
Responsible for running containers on the node
(b)/var/log/kube-proxy.log
Responsible for service load balancing
```
9. More reading:https://kubernetes.io/docs/tasks/debug-application-cluster/\debug-service/andhttps:
    //kubernetes.io/docs/tasks/debug-application-cluster/\determine-reason-pod-failure/

### Exercise 12.2: Viewing Logs Output

Container standard out can be seen via the **kubectl logs** command. If there is no standard out, you would not see any output.
In addition, the logs would be destroyed if the container is destroyed.

1. View the current Pods in the cluster. Be sure to view Pods in all namespaces.

```
student@lfs458-node-1a0a:~$ kubectl get po --all-namespaces
NAMESPACE NAME READY STATUS RESTARTS AGE
default ds-one-qc72k 1/1 Running 0 3h
default ds-one-z31r4 1/1 Running 0 3h
....
kube-system etcd-lfs458-node-1a0a 1/1 Running 2 9h
```

#### 12.1. LABS 91

```
kube-system kube-apiserver-lfs458-node-1a0a 1/1 Running 2 9h
kube-system kube-controller-manager-lfs458-node-1a0a 1/1 Running 2 9h
kube-system kube-dns-2425271678-w80vx 3/3 Running 6 9h
kube-system kube-scheduler-lfs458-node-1a0a 1/1 Running 2 9h
```
```
...
```
2. View the logs associated with various infrastructure pods. Using the **Tab** key you can get a list and choose a container.
    Then you can start typing the name of a pod and use **Tab** to complete the name.

```
student@lfs458-node-1a0a:~$ kubectl -n kube-system logs <Tab><Tab>
calico-etcd-n6h2q
etcd-lfs458-1-11-1update-cm35
calico-kube-controllers-74b888b647-9ds42
kube-apiserver-lfs458-1-11-1update-cm35
calico-node-6j8hc
kube-controller-manager-lfs458-1-11-1update-cm35
calico-node-dq6kf
kube-proxy-8sn6f
coredns-78fcdf6894-7fpfp
kube-proxy-wf5dr
coredns-78fcdf6894-g6k99
kube-scheduler-lfs458-1-11-1update-cm35
```
```
student@lfs458-node-1a0a:~$ kubectl -n kube-system logs \
kube-apiserver-lfs458-1-11-1update-cm35
Flag --insecure-port has been deprecated, This flag will be
removed in a future version.
I0729 21:29:23.026394 1 server.go:703] external host
was not specified, using 10.128.0.2
I0729 21:29:23.026667 1 server.go:145] Version: v1.11.1
I0729 21:29:23.784000 1 plugins.go:158] Loaded 8 mutating
admission controller(s) successfully in the following order:
NamespaceLifecycle,LimitRanger,ServiceAccount,NodeRestriction,
Priority,DefaultTolerationSeconds,DefaultStorageClass,
MutatingAdmissionWebhook.
I0729 21:29:23.784025 1 plugins.go:161] Loaded 6 validating
admission controller(s) successfully in the following order:
LimitRanger,ServiceAccount,Priority,PersistentVolumeClaimResize,
ValidatingAdmissionWebhook,ResourceQuota.
<output_omitted>
```
3. View the logs of other Pods in your cluster.


#### 92 CHAPTER 12. LOGGING AND TROUBLESHOOTING


**Chapter 13**

**Custom Resource Definition**

### 13.1 Labs

### Exercise 13.1: Create a Custom Resource Definition

### Overview

ThirdPartyResourceis no longer included with the API inv1.8and its use will return a validation error. If you have upgraded
from a version prior to Kubernetesv1.7, you will need to convert them toCustomResourceDefinitions (CRD). A new
resource often requires a controller to manage the resource. Creation of the controller is beyond the scope of this course,
basically it is a watch-loop comparing a spec file to the current state and making changes until the states match. A good
discussion of creating a controller can be found here:https://coreos.com/blog/introducing-operators.html.

### Create a Custom Resource Definition

We will make a simple CRD, but without any particular action. It will be enough to find the object ingested into the API and
responding to commands.

1. We will create a new YAML file.

```
student@lfs458-node-1a0a:~$ vim crd.yaml
```
```
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
name: crontabs.training.lfs458.com
# This name must match names below.
# <plural>.<group> syntax
spec:
scope: Cluster #Could also be Namespaced
group: training.lfs458.com
```
#### 93


#### 94 CHAPTER 13. CUSTOM RESOURCE DEFINITION

```
version: v1
names:
kind: CronTab #Typically CamelCased for resource manifest
plural: crontabs #Shown in URL
singular: crontab #Short name for CLI alias
shortNames:
```
- ct #CLI short name
2. Add the new resource to the cluster.

```
student@lfs458-node-1a0a:~$ kubectl create -f crd.yaml
customresourcedefinition.apiextensions.k8s.io/crontabs.training.lfs458.com
created
```
3. View and describe the resource. You’ll note the **describe** output is unlike other objects we have seen so far.

```
student@lfs458-node-1a0a:~$ kubectl get crd
NAME CREATED AT
crontabs.training.lfs458.com 2018-08-03T05:25:20Z
<output_omitted>
```
```
student@lfs458-node-1a0a:~$ kubectl describe crd crontab<Tab>
Name: crontabs.training.lfs458.com
Namespace:
Labels: <none>
Annotations: <none>
API Version: apiextensions.k8s.io/v1beta1
Kind: CustomResourceDefinition
<output_omitted>
```
4. Now that we have a new API resource we can create a new object of that type. In this case it will be a crontab-like
    image, which does not actually exist, but is being used for demonstration.

```
student@lfs458-node-1a0a:~$ vim new-crontab.yaml
```
```
apiVersion: "training.lfs458.com/v1"
# This is from the group and version of new CRD
kind: CronTab
# The kind from the new CRD
metadata:
name: new-cron-object
spec:
cronSpec: "*/5 * * * *"
image: some-cron-image
#Does not exist
```
5. Create the new object and view the resource using short and long name.

```
student@lfs458-node-1a0a:~$ kubectl create -f new-crontab.yaml
crontab.training.lfs458.com/new-cron-object created
```
```
student@lfs458-node-1a0a:~$ kubectl get CronTab
NAME AGE
new-cron-object 22s
```
```
student@lfs458-node-1a0a:~$ kubectl get ct
NAME AGE
new-cron-object 29s
```
```
student@lfs458-node-1a0a:~$ kubectl describe ct
```

#### 13.1. LABS 95

```
Name: new-cron-object
Namespace:
Labels: <none>
```
```
<output_omitted>
```
```
Spec:
Cron Spec: */5 * * * *
Image: some-cron-image
Events: <none>
```
6. To clean up the resources we will delete theCRD. This should delete all of the endpoints and objects using it as well.

```
student@lfs458-node-1a0a:~$ kubectl delete -f crd.yaml
customresourcedefinition.apiextensions.k8s.io
"crontabs.training.lfs458.com" deleted
```
```
student@lfs458-node-1a0a:~$ kubectl get ct
Error from server (NotFound): Unable to list "crontabs": the server
could not find the requested resource
(get crontabs.training.lfs458.com)
```

#### 96 CHAPTER 13. CUSTOM RESOURCE DEFINITION


**Chapter 14**

**Kubernetes Federation**

### 14.1 Labs

There is no lab to complete for this chapter.

#### 97


#### 98 CHAPTER 14. KUBERNETES FEDERATION


**Chapter 15**

**Helm**

### 15.1 Labs

### Exercise 15.1: Working with Helm and Charts

### Overview

**helm** allows for easy deployment of complex configurations. This could be handy for a vendor to deploy a multi-part application
in a single step. Through the use of aChart, or template file, the required components and their relationships are declared.
Local agents like **Tiller** use the API to create objects on your behalf. Effectively its orchestration for orchestration.

There are a few ways to install **Helm**. The newest version may require building from source code. We will download a recent,
stable version. Once installed we will deploy aChart, which will configure **Hadoop** on our cluster.

### Install Helm

1. On themasternode use **wget** to download the compressed tar file. The short URL below is for:https://storage.
    googleapis.com/kubernetes-helm/helm-v2.7.0-linux-amd64.tar.gz

```
student@lfs458-node-1a0a:~$ wget goo.gl/nbEcHn
<output_omitted>
nbEcHn 100%[====================>] 11.61M --.-KB/s in 0.1s
```
```
2018-08-03 05:34:56 (91.7 MB/s) - nbEcHn saved [12169373/12169373]
```
2. Uncompress and expand the file.

```
student@lfs458-node-1a0a:~$ tar -xvf nbEcHn
linux-amd64/
linux-amd64/README.md
linux-amd64/helm
linux-amd64/LICENSE
```
#### 99


#### 100 CHAPTER 15. HELM

3. Copy the **helm** binary to the/usr/local/bin/directory, so it is usable via the shell search path.

```
student@lfs458-node-1a0a:~$ sudo cp linux-amd64/helm /usr/local/bin/
```
4. Due to new **RBAC** configuration **helm** is unable to run in thedefaultnamespace, in this version of Kubernetes. During
    initialization you could choose to create and declare a new namespace. Other RBAC issues may be encountered even
    then. In this lab we will create a service account for **tiller** , and give it admin abilities on the cluster. More on **RBAC** in
    another chapter.
    Begin by creating theserviceaccountobject.

```
student@lfs458-node-1a0a:~$ kubectl create serviceaccount \
--namespace kube-system tiller
serviceaccount "tiller" created
```
5. Bind theserviceaccountto the admin role calledcluster-admininside thekube-systemnamespace.

```
student@lfs458-node-1a0a:~$ kubectl create clusterrolebinding \
tiller-cluster-rule \
--clusterrole=cluster-admin \
--serviceaccount=kube-system:tiller
clusterrolebinding.rbac.authorization.k8s.io/tiller-cluster-rule created
```
6. We can now initialize **helm**. This process will also configure **tiller** the client process. There are several possible options
    to pass such asnodeAffinity, a particular version of software, alternate storage backend, and even a dry-run option
    to generate JSON or YAML output. The output could be edited and ingested into **kubectl**. We will use default values in
    this case.

```
student@lfs458-node-1a0a:~$ helm init
```
```
<output_omitted>
```
7. Update thetiller-deploydeployment to have the service account.

```
student@lfs458-node-1a0a:~$ kubectl -n kube-system patch deployment \
tiller-deploy -p \
’{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}’
deployment.extensions/tiller-deploy patched
```
8. Verify the **tiller** pod is running. Examine the logs of the pod. Note that each line of log begins with an tag of the
    component generating the messages, such as[main],[storage], and[storage].

```
student@lfs458-node-1a0a:~$ kubectl get pods --all-namespaces
<output_omitted>
kube-system tiller-deploy-84b97f465c-76lvs 1/1 Running 0 30m
```
```
student@lfs458-node-1a0a:~$ kubectl -n kube-system logs \
tiller-deploy-84b97f465c-76lvs
<output_omitted>
```
9. View the available sub-commands for **helm**. As with other Kubernetes tools, expect ongoing change.

```
student@lfs458-node-1a0a:~$ helm help
<output_omitted>
```
10. View the current configuration files, archives and plugins for helm. Return to this directory after you have worked with a
    Chartlater in the lab.


#### 15.1. LABS 101

```
student@lfs458-node-1a0a:~$ helm home
/home/student/.helm
```
```
student@lfs458-node-1a0a:~$ ls -R /home/student/.helm/
/home/student/.helm/:
cache plugins repository starters
```
```
/home/student/.helm/cache:
archive
<output_omitted>
```
11. Verify **helm** and **tiller** are responding, also check the current version installed.

```
student@lfs458-node-1a0a:~$ helm version
Client: &version.Version{SemVer:"v2.7.0", GitCommit:"08c1144f5...
Server: &version.Version{SemVer:"v2.7.0", GitCommit:"08c1144f5...
```
12. Ensure both are upgraded to the most recent stable version.

```
student@lfs458-node-1a0a:~$ helm init --upgrade
$HELM_HOME has been configured at /home/student/.helm.
```
```
Tiller (the Helm server-side component) has been upgraded
to the current version.
Happy Helming!
```
13. AChartis a collection of containers to deploy an application. There is a collection available onhttps://github.com/
    kubernetes/charts/tree/master/stable, provided by vendors, or you can make your own. Take a moment and
    view the current stableCharts. Then search for available stable databases.

```
student@lfs458-node-1a0a:~$ helm search database
NAME VERSION DESCRIPTION
stable/cockroachdb 2.0.3 CockroachDB is a scalable, survivable,...
stable/dokuwiki 3.3.0 DokuWiki is a standards-compliant, ...
stable/janusgraph 0.2.0 Open source, scalable graph database.
stable/kubedb 0.1.3 DEPRECATED KubeDB by AppsCode - Making...
stable/mariadb 5.2.3 Fast, reliable, scalable, and easy to use...
<output_omitted>
```
14. We will install the **mariadb**. Take a look at install detailshttps://github.com/kubernetes/charts/tree/master/
    stable/mariadb#custom-mycnf-configurationThe **–debug** option will create a lot of output. Note the interesting
    name for the deployment, likeillmannered-salamander. The output will typically suggest ways to access the software.
    As well we will indicate that we do not want persistent storage, which would require use to create an availablePV.

```
student@lfs458-node-1a0a:~$ helm --debug install stable/mariadb \
--set master.persistence.enabled=false \
--set slave.persistence.enabled=false
```
```
[debug] Created tunnel using local port: ’38396’
```
```
[debug] SERVER: "localhost:38396"
```
```
[debug] Original chart version: ""
[debug] Fetched stable/mariadb to /home/student/.helm/cache/archive/mar...
```
```
[debug] CHART PATH: /home/student/.helm/cache/archive/mariadb-2.0.1.tgz
```
```
NAME: illmannered-salamander
<output_omitted>
```

#### 102 CHAPTER 15. HELM

15. Using some of the information at the end of the previous command output we will deploy another container and access
    the database. We begin by getting the root password forillmannered-salamander. Be aware the output lacks a
    carriage return, so the next prompt will appear on the same line. We will need the password to access the running
    MariaDBdatabase.

```
student@lfs458-node-1a0a:~$ kubectl get secret -n default \
illmannered-salamander-mariadb \
-o jsonpath="{.data.mariadb-root-password}" \
| base64 --decode
IFBldzAQfx
```
16. Now we will install another container to act as a client for the database. We will use **apt-get** to install client software.

```
student@lfs458-node-1a0a:~$ kubectl run -i --tty ubuntu \
--image=ubuntu:16.04 --restart=Never -- bash -il
If you don’t see a command prompt, try pressing enter.
root@ubuntu:/#
```
```
root@ubuntu:/# apt-get update ; apt-get install -y mariadb-client
<output_omitted>
```
17. Use the client software to access the database. The following command uses the server name and the root password
    we found in a previous step. Both of yours will be different.

```
root@ubuntu:/# mysql -h illmannered-salamander-mariadb -p
Enter password: IFBldzAQfx
Welcome to the MariaDB monitor. Commands end with ; or \g.
Your MariaDB connection id is 153
Server version: 10.1.28-MariaDB Source distribution
```
```
Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.
```
```
Type ’help;’ or ’\h’ for help. Type ’\c’ to clear the current input statement.
```
```
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database |
+--------------------+
| information_schema |
| my_database |
| mysql |
| performance_schema |
| test |
+--------------------+
5 rows in set (0.00 sec)
```
```
MariaDB [(none)]>
MariaDB [(none)]> quit
root@ubuntu:/# exit
```
18. View theCharthistory on the system. The use of the **-a** option will show allChartsincludingdeletedandfailed
    attempts. The output below shows the current runningChartas well as a previously deleted **hadoop** Chart.

```
student@lfs458-node-1a0a:~$ helm list -a
NAME REVISION UPDATED STATUS CHART NAMESPACE
goodly-beetle 1 Wed Nov 8 23:01:24 2017 DELETED hadoop-1.0.1 default
illmannered-salamander 1 Thu Nov 9 05:00:12 2017 DEPLOYED mariadb-...
```
19. Delete the **mariadb** Chart. No output should happen from the list.


#### 15.1. LABS 103

```
student@lfs458-node-1a0a:~$ helm delete illmannered-salamander
release "illmannered-salamander" deleted
```
```
student@lfs458-node-1a0a:~$ helm list
```
20. Add another repository and view theChartsavailable.

```
student@lfs458-node-1a0a:~$ helm repo add common \
http://storage.googleapis.com/kubernetes-charts
"common" has been added to your repositories
```
```
student@lfs458-node-1a0a:~$ helm repo list
NAME URL
stable https://kubernetes-charts.storage.googleapis.com
local http://127.0.0.1:8879/charts
common http://storage.googleapis.com/kubernetes-charts
```
```
student@lfs458-node-1a0a:~$ helm search
NAME VERSION DESCRIPTION
stable/acs-engine-autoscaler 2.1.0 Scales worker nodes within...
stable/artifactory 6.2.0 Universal Repository Manag...
<output_omitted>
```

#### 104 CHAPTER 15. HELM


**Chapter 16**

**Security**

### 16.1 Labs

### Exercise 16.1: Working with TLS

### Overview

We have learned that the flow of access to a cluster begins with TLS connectivity, then authentication followed by authorization,
finally an admission control plug-in allows advanced features prior to the request being fulfilled. The use ofInitializers
allows the flexibility of a shell-script to dynamically modify the request. As security is an important, ongoing concern, there
may be multiple configurations used depending on the needs of the cluster.

Every process making API requests to the cluster must authenticate or be treated as an anonymous user.

### Working with TLS

While one can have multiple cluster root Certificate Authorities (CA) by default each cluster uses their own, intended for intra-
cluster communication. The CA certificate bundle is distributed to each node and as a secret to default service accounts. The
**kubelet** is a local agent which ensures local containers are running and healthy.

1. View the **kubelet** on both the master and secondary nodes. The **kube-apiserver** also shows security information such
    as certificates and authorization mode. As **kubelet** is a **systemd** service we will start looking at that output.

```
student@lfs458-node-1a0a:~$ systemctl status kubelet.service
kubelet.service - kubelet: The Kubernetes Node Agent
Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: en
Drop-In: /etc/systemd/system/kubelet.service.d
10-kubeadm.conf
<output_omitted>
```
2. If we look at the status output, and follow the cgroup information, which is a long line we where configuration settings
    are drawn from, we see where the configuration file can be found.

#### 105


#### 106 CHAPTER 16. SECURITY

```
CGroup: /system.slice/kubelet.service
19523 /usr/bin/kubelet .... --config=/var/lib/kubelet/config.yaml ..
```
3. Take a look at the settings in the/var/lib/kubelet/config.yamlfile. Among other information we can see the
    /etc/kubernetes/pki/directory is used for accessing the **kube-apiserver**. Near the end of the output it also sets the
    directory to find other pod spec files.

```
student@lfs458-node-1a0a:~$ sudo less /var/lib/kubelet/config.yaml
address: 0.0.0.0
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
anonymous:
enabled: false
webhook:
cacheTTL: 2m0s
enabled: true
x509:
clientCAFile: /etc/kubernetes/pki/ca.crt
```
4. Other agents on the master node interact with the **kube-apiserver**. View the configuration files where these settings
    are made. This was set in the previous YAML file. Look at one of the files for cert information.

```
student@lfs458-node-1a0a:~$ sudo ls /etc/kubernetes/manifests/
etcd.yaml kube-controller-manager.yaml
kube-apiserver.yaml kube-scheduler.yaml
```
```
student@lfs458-node-1a0a:~$ sudo less \
/etc/kubernetes/manifests/kube-controller-manager.yaml
<output_omitted>
```
5. The use oftokenshas become central to authorizing component communication. The tokens are kept as **secrets**. Take
    a look at the current secrets in thekube-systemnamespace.

```
student@lfs458-node-1a0a:~$ kubectl -n kube-system get secrets
NAME TYPE
DATA AGE
attachdetach-controller-token-xqr8n kubernetes.io/service-account-token
3 5d
bootstrap-signer-token-xbp6s kubernetes.io/service-account-token
3 5d
bootstrap-token-i3r13t bootstrap.kubernetes.io/token
7 5d
<output_omitted>
```
6. Take a closer look at one of the secrets and the token within. Thecertificate-controller-tokencould be one to
    look at. The use of the Tab key can help with long names. Long lines have been truncated in the output below.

```
student@lfs458-node-1a0a:~$ kubectl -n kube-system get secrets \
certificate<Tab> -o yaml
apiVersion: v1
data:
ca.crt: LS0tLS1CRUdJTi.....
namespace: a3ViZS1zeXN0ZW0=
token: ZXlKaGJHY2lPaUpTVXpJM....
kind: Secret
metadata:
annotations:
kubernetes.io/service-account.name: certificate-controller
kubernetes.io/service-account.uid: 7dfa2aa0-9376-11e8-8cfb
-42010a800002
creationTimestamp: 2018-07-29T21:29:36Z
name: certificate-controller-token-wnrwh
```

#### 16.1. LABS 107

```
namespace: kube-system
resourceVersion: "196"
selfLink: /api/v1/namespaces/kube-system/secrets/certificate-
controller-token-wnrwh
uid: 7dfbb237-9376-11e8-8cfb-42010a800002
type: kubernetes.io/service-account-token
```
7. The **kubectl config** command can also be used to view and update parameters. When making updates this could avoid
    a typo removing access to the cluster. View the current configuration settings. The keys and certs are redacted from the
    output automatically.

```
student@lfs458-node-1a0a:~$ kubectl config view
apiVersion: v1
clusters:
```
- cluster:
    certificate-authority-data: REDACTED
<output_omitted>
8. View the options, such as setting a password for the admin instead of a key. Read through the examples and options.

```
student@lfs458-node-1a0a:~$ kubectl config set-credentials -h
Sets a user entry in kubeconfig
<output_omitted>
```
9. Make a copy of your access configuration file. Later steps will update this file and we can view the differences.

```
student@lfs458-node-1a0a:~$ cp ~/.kube/config ~/cluster-api-config
```
10. Explore working with cluster and security configurations both using **kubectl** and **kubeadm**. Among other values, find
    the name of your cluster. You will need to becomerootto work with **kubeadm**.

```
student@lfs458-node-1a0a:~$ kubectl config <Tab><Tab>
current-context get-contexts set-context view
delete-cluster rename-context set-credentials
delete-context set unset
get-clusters set-cluster use-context
```
```
student@lfs458-node-1a0a:~$ sudo -i
```
```
root@lfs458-node-1a0a:~# kubeadm token -h
<output_omitted>
```
```
root@lfs458-node-1a0a:~# kubeadm config -h
<output_omitted>
```
11. Review the cluster default configuration settings. At over 150 lines there may be some interesting tidbits to the security
    and infrastructure of the cluster.

```
student@lfs458-node-1a0a:~$ kubeadm config print-default
api:
advertiseAddress: 10.128.0.2
bindPort: 6443
controlPlaneEndpoint: ""
apiVersion: kubeadm.k8s.io/v1alpha2
auditPolicy:
logDir: /var/log/kubernetes/audit
logMaxAge: 2
path: ""
bootstrapTokens:
```
- groups:
    - system:bootstrappers:kubeadm:default-node-token
    token: abcdef.0123456789abcdef
<output_omitted>


#### 108 CHAPTER 16. SECURITY

### Exercise 16.2: Authentication and Authorization

Kubernetes clusters have to types of usersservice accountsandnormal users, butnormal usersare assumed to be
managed by an outside service. There are no objects to represent them and they cannot be added via an API call, butservice
accountscan be added.

We will use **RBAC** to configure access to actions within anamespacefor a new contractor,Developer Danwho will be working
on a new project.

1. Create two namespaces, one forproductionand the other fordevelopment.

```
student@lfs458-node-1a0a:~$ kubectl create ns development
namespace "development" created
```
```
student@lfs458-node-1a0a:~$ kubectl create ns production
namespace "production" created
```
2. View the current clusters and context available. The context allows you to configure the cluster to use, namespace and
    user for **kubectl** commands in an easy and consistent manner.

```
student@lfs458-node-1a0a:~$ kubectl config get-contexts
CURRENT NAME CLUSTER AUTHINFO NAMESPACE
* kubernetes-admin@kubernetes kubernetes kubernetes-admin
```
3. Create a new userDevDanand assign a password oflfs458.

```
student@lfs458-node-1a0a:~$ sudo useradd -s /bin/bash DevDan
student@lfs458-node-1a0a:~$ sudo passwd DevDan
Enter new UNIX password: lfs458
Retype new UNIX password: lfs458
passwd: password updated successfully
```
4. Generate a private key then Certificate Signing Request (CSR) forDevDan.

```
student@lfs458-node-1a0a:~$ openssl genrsa -out DevDan.key 2048
Generating RSA private key, 2048 bit long modulus
......+++
.........+++
e is 65537 (0x10001)
```
```
student@lfs458-node-1a0a:~$ openssl req -new -key DevDan.key \
-out DevDan.csr -subj "/CN=DevDan/O=development"
```
5. Using thew newly created request generate a self-signed certificate using the x509 protocol. Use the CA keys for the
    Kubernetes cluster and set a 45 day expiration. You’ll need to use **sudo** to access to the inbound files.

```
student@lfs458-node-1a0a:~$ sudo openssl x509 -req -in DevDan.csr \
-CA /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key \
-CAcreateserial \
-out DevDan.crt -days 45
Signature ok
subject=/CN=DevDan/O=development
Getting CA Private Key
```
6. Update the access config file to reference the new key and certificate. Normally we would move them to a safe directory
    instead of a non-root user’s home.

```
student@lfs458-node-1a0a:~$ kubectl config set-credentials DevDan \
--client-certificate=/home/student/DevDan.crt \
--client-key=/home/student/DevDan.key
User "DevDan" set.
```

#### 16.1. LABS 109

7. View the update to your credentials file. Use **diff** to compare against the copy we made earlier.

```
student@lfs458-node-1a0a:~$ diff cluster-api-config .kube/config
9a10,14
> namespace: development
> user: DevDan
> name: DevDan-context
> - context:
> cluster: kubernetes
15a21,25
> - name: DevDan
> user:
> as-user-extra: {}
> client-certificate: /home/student/DevDan.crt
> client-key: /home/student/DevDan.key
```
8. We will now create a context. For this we will need the name of the cluster, namespace and CN of the user we set or
    saw in previous steps.

```
student@lfs458-node-1a0a:~$ kubectl config set-context DevDan-context \
--cluster=kubernetes \
--namespace=development \
--user=DevDan
Context "DevDan-context" created.
```
9. Attempt to view thePodsinside theDevDan-context. Be aware you will get an error.

```
student@lfs458-node-1a0a:~$ kubectl --context=DevDan-context get pods
Error from server (Forbidden): pods is forbidden: User "DevDan"
cannot list pods in the namespace "development"
```
10. Verify the context has been properly set.

```
student@lfs458-node-1a0a:~$ kubectl config get-contexts
CURRENT NAME CLUSTER AUTHINFO NAMESPACE
DevDan-context kubernetes DevDan development
* kubernetes-admin@kubernetes kubernetes kubernetes-admin
```
11. Again check the recent changes to the cluster access config file.

```
student@lfs458-node-1a0a:~$ diff cluster-api-config .kube/config
9a10,14
> namespace: development
> user: DevDan
> name: DevDan-context
> - context:
> cluster: kubernetes
15a21,25
> - name: DevDan
> user:
<output_omitted>
```
12. We will now create a YAML file to associate RBAC rights to a particular namespace andRole.

```
student@lfs458-node-1a0a:~$ vim role-dev.yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
namespace: development
name: developer
rules:
```
- apiGroups: ["", "extensions", "apps"]
    resources: ["deployments", "replicasets", "pods"]
    verbs: ["list", "get", "watch", "create", "update", "patch", "delete"]
# You can use ["*"] for all verbs


#### 110 CHAPTER 16. SECURITY

13. Create the object. Check white space and for typos if you encounter errors.

```
student@lfs458-node-1a0a:~$ kubectl create -f role-dev.yaml
role.rbac.authorization.k8s.io/developer created
```
14. Now we create aRoleBindingto associate theRolewe just created with a user. Create the object when the file has
    been created.

```
student@lfs458-node-1a0a:~$ vim rolebind.yaml
```
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
name: developer-role-binding
namespace: development
subjects:
```
- kind: User
    name: DevDan
    apiGroup: ""
roleRef:
    kind: Role
    name: developer
    apiGroup: ""

```
student@lfs458-node-1a0a:~$ kubectl apply -f rolebind.yaml
rolebinding.rbac.authorization.k8s.io/developer-role-binding created
```
15. Test the context again. This time it should work. There are noPodsrunning so you should get a response ofNo
    resources found.

```
student@lfs458-node-1a0a:~$ kubectl --context=DevDan-context get pods
No resources found.
```
16. Create a new pod, verify it exists, then delete it.

```
student@lfs458-node-1a0a:~$ kubectl --context=DevDan-context \
create deployment nginx --image=nginx
deployment.apps/nginx created
```
```
student@lfs458-node-1a0a:~$ kubectl --context=DevDan-context get pods
NAME READY STATUS RESTARTS AGE
nginx-7c87f569d-7gb9k 1/1 Running 0 5s
```
```
student@lfs458-node-1a0a:~$ kubectl --context=DevDan-context delete deploy nginx
deployment.extensions "nginx" deleted
```
17. We will now create a different context for production systems. TheRolewill only have the ability to view, but not create
    or delete resources. Begin by copying and editing theRoleandRoleBindingsYAML files.

```
student@lfs458-node-1a0a:~$ cp role-dev.yaml role-prod.yaml
```
```
student@lfs458-node-1a0a:~$ vim role-prod.yaml
```
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
namespace: production #<<- This line
name: dev-prod #<<- and this line
rules:
```
- apiGroups: ["", "extensions", "apps"]
    resources: ["deployments", "replicasets", "pods"]
    verbs: ["get", "list", "watch"] #<<- and this one


#### 16.1. LABS 111

```
student@lfs458-node-1a0a:~$ cp rolebind.yaml rolebindprod.yaml
```
```
student@lfs458-node-1a0a:~$ vim rolebindprod.yaml
```
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
name: production-role-binding
namespace: production
subjects:
```
- kind: User
    name: DevDan
    apiGroup: ""
roleRef:
    kind: Role
    name: dev-prod
    apiGroup: ""
18. Create both new objects.

```
student@lfs458-node-1a0a:~$ kubectl apply -f role-prod.yaml
role.rbac.authorization.k8s.io/dev-prod created
```
```
student@lfs458-node-1a0a:~$ kubectl apply -f rolebindprod.yaml
rolebinding.rbac.authorization.k8s.io/production-role-binding created
```
19. Create the new context for production use.

```
student@lfs458-node-1a0a:~$ kubectl config set-context ProdDan-context \
--cluster=kubernetes \
--namespace=production \
--user=DevDan
Context "ProdDan-context" created.
```
20. Verify that user DevDan can view pods using the new context.

```
student@lfs458-node-1a0a:~$ kubectl --context=ProdDan-context get pods
No resources found.
```
21. Try to create aPodin production. The developer should beForbidden.

```
student@lfs458-node-1a0a:~$ kubectl --context=ProdDan-context run \
nginx --image=nginx
Error from server (Forbidden): deployments.extensions is forbidden: User "DevDan" cannot \
create deployments.extensions in the namespace "production"
```
22. View the details of a role.

```
student@lfs458-node-1a0a:~$ kubectl describe role dev-prod -n production
Name: dev-prod
Labels: <none>
Annotations: kubectl.kubernetes.io/last-applied-configuration=
{"apiVersion":"rbac.authorization.k8s.io/v1beta1","kind":"Role"
,"metadata":{"annotations":{},"name":"dev-prod","namespace":
"production"},"rules":[{"api...
PolicyRule:
Resources Non-Resource URLs Resource Names Verbs
--------- ----------------- -------------- -----
deployments [] [] [get list watch]
deployments.apps [] [] [get list watch]
<output_omitted>
```

#### 112 CHAPTER 16. SECURITY

23. Experiment with other subcommands in both contexts. They should match those listed in the respective roles.

### Exercise 16.3: Admission Controllers

The last stop before a request is sent to the API server is anadmission control plug-in. They interact with features such
as setting parameters like a default storage class, checking resource quotas, or security settings. A newer feature (v1.7.x) is
dynamic controllers which allow new controllers to be ingested or configured at runtime.

1. View the currentadmission controllersettings. Unlike earlier versions of Kubernetes the controllers are now com-
    piled into the server, instead of being passed at run-time. Instead of a list of which controllers to use we can enable and
    disable specific plugins.

```
student@lfs458-node-1a0a:~$ sudo grep admission \
/etc/kubernetes/manifests/kube-apiserver.yaml
```
- --disable-admission-plugins=PersistentVolumeLabel
- --enable-admission-plugins=NodeRestriction


