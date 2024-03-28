# CKA-Simulator-Practice-Test-Killer-Sh

# Prerequisites
locate .bashrc

export do=’—dry-run=client -o yaml’
export now="--force --grace-period 0"

# set numbers in vi

vi ~/.vimrc

set mouse=r

# Q1

You have access to multiple clusters from your main terminal through kubectl contexts. Write all those context names into /opt/course/1/contexts.

Next write a command to display the current context into /opt/course/1/context_default_kubectl.sh, the command should use kubectl.

Finally write a second command doing the same thing into /opt/course/1/context_default_no_kubectl.sh, but without the use of kubectl.

---

CKA-Simulator-Practice-Test-Killer-Sh

Context using kubectl and no kubectl commands
```
kubectl config get-contexts

kubectl config get-contexts --no-headers | awk {'print $2'} > /opt/course/1/contexts
OR
k config get-contexts -o name > /opt/course/1/contexts
kubectl config get-contexts --output=name > /opt/course/1/contexts

echo "kubectl config current-context" > /opt/course/1/context_default_kubectl.sh
bash /opt/course/1/context_default_kubectl.sh

echo "cat .kube/config | grep -i current-context" > /opt/course/1/context_default_no_kubectl.sh
bash /opt/course/1/context_default_no_kubectl.sh 
 
```

# Q2
Create a single Pod of image httpd:2.4.41-alpine in Namespace default. The Pod should be named pod1 and the container should be named pod1-container. This Pod should only be scheduled on controlplane nodes. Do not add new labels to any nodes.

---


Pod-Node Affinity
```
alias k=kubectl

kubectl config use-context k8s-c1-H

k get nodes --show-labels

k run pod1 --image=httpd:2.4.41-alpine --dry-run=client -o yaml >> q2.yaml
vim q2.yaml
//add nodeName
```

# Q3

There are two Pods named o3db-* in Namespace project-c13. C13 management asked you to scale the Pods down to one replica to save resources.

---

Scaling Replicas
```

k -n project-c13 get deploy,ds,sts | grep o3db

k -n project-c13 scale statefulset 03db --replicas=1
```
# Q4

Do the following in Namespace default. Create a single Pod named ready-if-service-ready of image nginx:1.16.1-alpine. Configure a LivenessProbe which simply executes command true. Also configure a ReadinessProbe which does check if the url http://service-am-i-ready:80 is reachable, you can use wget -T2 -O- http://service-am-i-ready:80 for this. Start the Pod and confirm it isn't ready because of the ReadinessProbe.

Create a second Pod named am-i-ready of image nginx:1.16.1-alpine with label id: cross-server-ready. The already existing Service service-am-i-ready should now have that second Pod as endpoint.

Now the first Pod should be in ready state, confirm that.

---

Liveness Probe/Readiness Probe
```
kubectl config use-context k8s-c1-H

k run ready-if-service-ready --image=nginx:1.16.1-alpine --dry-run=client -o yaml > q4pod1.yaml
k apply -f q4pod1.yaml 

k run am-i-ready --image=nginx:1.16.1-alpine --labels=id=cross-server-ready
k describe svc service-am-i-ready
```

# Q5

There are various Pods in all namespaces. Write a command into /opt/course/5/find_pods.sh which lists all Pods sorted by their AGE (metadata.creationTimestamp).

Write a second command into /opt/course/5/find_pods_uid.sh which lists all Pods sorted by field metadata.uid. Use kubectl sorting for both commands.

---

Use full command (kubectl) for shell scripts 

```
echo "kubectl get pod -A --sort-by=.metadata.creationTimestamp" > /opt/course/5/find_pods.sh
bash find_pods.sh

echo "kubectl get pod -A --sort-by=.metadata.uid" > find_pods_uid.sh
bash find_pods_uid.sh 
```

# Q6

Create a new PersistentVolume named safari-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

Next create a new PersistentVolumeClaim in Namespace project-tiger named safari-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

Finally create a new Deployment safari in Namespace project-tiger which mounts that volume at /tmp/safari-data. The Pods of that Deployment should be of image httpd:2.4.41-alpine.

---

Create PV , PVC , Deployment 

```
k create -f q6pv.yaml 
k create -f q6pvc.yaml 
k get pv,pvc -n project-tiger 

k create deployment safari --image=httpd:2.4.41-alpine -n project-tiger --dry-run=client -o yaml > q6dep.yaml
k get pv,pvc,deployments.apps -n project-tiger 
```

# Q7

The metrics-server has been installed in the cluster. Your college would like to know the kubectl commands to:

show Nodes resource usage
show Pods and their containers resource usage
Please write the commands into /opt/course/7/node.sh and /opt/course/7/pod.sh.

---

kubectl config use-context k8s-c1-H

Install Metrics Server :
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Fix Error:
```
k edit deployments.apps -n kube-system 
```
![image](https://user-images.githubusercontent.com/54164634/190094549-015cc38a-87cb-4a98-ab0d-3c8920c1d8ef.png)

```
k get deployments.apps -n kube-system 
kubectl top nodes
```

```
echo "kubectl top nodes" > /opt/course/7/node.sh
bash /opt/course/7/node.sh
```
```
echo "kubectl top pods" > /opt/course/7/pod.sh
bash /opt/course/7/pod.sh
```

# Q8

Ssh into the controlplane node with ssh cluster1-controlplane1. Check how the controlplane components kubelet, kube-apiserver, kube-scheduler, kube-controller-manager and etcd are started/installed on the controlplane node. Also find out the name of the DNS application and how it's started/installed on the controlplane node.

Write your findings into file /opt/course/8/controlplane-components.txt. The file should be structured like:

# /opt/course/8/controlplane-components.txt
kubelet: [TYPE]
kube-apiserver: [TYPE]
kube-scheduler: [TYPE]
kube-controller-manager: [TYPE]
etcd: [TYPE]
dns: [TYPE] [NAME]
Choices of [TYPE] are: not-installed, process, static-pod, pod

---

Identify PODs, Static PODs, Processes etc.

```
ssh cluster1-master1
k get pod -n kube-system
k get deploy -n kube-system 
k get all -n kube-system 
```
```
ps -ef | grep -i kubelet
cd /etc/kubernetes/manifests/
```


//output of /opt/course/8/master-components.txt

@**kubelet**: [process]
@**kube-apiserver**: [static-pod]
@**kube-scheduler**: [static-pod]
@**kube-controller-manager**: [static-pod]
@**etcd**: [static-pod]
@**dns**: [pod][coredns]


# Q9

Ssh into the controlplane node with ssh cluster2-controlplane1. Temporarily stop the kube-scheduler, this means in a way that you can start it again afterwards.

Create a single Pod named manual-schedule of image httpd:2.4-alpine, confirm it's created but not scheduled on any node.

Now you're the scheduler and have all its power, manually schedule that Pod on node cluster2-controlplane1. Make sure it's running.

Start the kube-scheduler again and confirm it's running correctly by creating a second Pod named manual-schedule2 of image httpd:2.4-alpine and check if it's running on cluster2-node1.

---

Test POD scheduling using kube-scheduler
```
kubectl config use-context k8s-c2-AC 
```
```
ssh cluster2-master1
```
```
k get pod -n kube-system 
cd /etc/kubernetes/manifests/
mv kube-scheduler.yaml /etc/kubernetes/
```

ADD: nodeName: cluster2-master1
```
k run manual-schedule --image=httpd:2.4-alpine
k edit pod manual-schedule
k replace --force -f /tmp/kubectl-edit-2160969323.yaml
```

```
mv /etc/kubernetes/kube-scheduler.yaml /etc/kubernetes/manifests/  
k run manual-schedule2 --image=httpd:2.4-alpine
```

# Q10

Create a new ServiceAccount processor in Namespace project-hamster. Create a Role and RoleBinding, both named processor as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace.

---

SA, Role, Rolebinding

```
k create sa processor -n project-hamster
k create role processor --resource=secrets,configmaps --verb=create -n project-hamster
k create rolebinding processor --role processor --serviceaccount=project-hamster:process -n project-hamster 
```

```
k get sa,role,rolebinding -n project-hamster
```

 
# Q11

Use Namespace project-tiger for the following. Create a DaemonSet named ds-important with image httpd:2.4-alpine and labels id=ds-important and uuid=18426a0b-5f59-4e10-923f-c0e078e82462. The Pods it creates should request 10 millicore cpu and 10 mebibyte memory. The Pods of that DaemonSet should run on all nodes, also controlplanes.

---

Create DaemonSet using template:
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

```
kubectl config use-context k8s-c1-H
```

```
k create -f q11.yaml 
k get ds -n project-tiger
```


# Q12

Use Namespace project-tiger for the following. Create a Deployment named deploy-important with label id=very-important (the Pods should also have this label) and 3 replicas. It should contain two containers, the first named container1 with image nginx:1.17.6-alpine and the second one named container2 with image google/pause.

There should be only ever one Pod of that Deployment running on one worker node. We have two worker nodes: cluster1-node1 and cluster1-node2. Because the Deployment has three replicas the result should be that on both nodes one Pod is running. The third Pod won't be scheduled, unless a new worker node will be added. Use topologyKey: kubernetes.io/hostname for this.

In a way we kind of simulate the behaviour of a DaemonSet here, but using a Deployment and a fixed number of replicas.

---

Deployment acting as DaemonSet (One Pod per Node)

```
k create deployment deploy-important --image=nginx:1.17.6-alpine -n project-tiger --replicas=3 --dry-run=client -o yaml > q12.yaml
k get pods -n project-tiger 
k replace --force -f q12.yaml
```
 
 
# Q13

Create a Pod named multi-container-playground in Namespace default with three containers, named c1, c2 and c3. There should be a volume attached to that Pod and mounted into every container, but the volume shouldn't be persisted or shared with other Pods.

Container c1 should be of image nginx:1.17.6-alpine and have the name of the node where its Pod is running available as environment variable MY_NODE_NAME.

Container c2 should be of image busybox:1.31.1 and write the output of the date command every second in the shared volume into file date.log. You can use while true; do date >> /your/vol/path/date.log; sleep 1; done for this.

Container c3 should be of image busybox:1.31.1 and constantly send the content of file date.log from the shared volume to stdout. You can use tail -f /your/vol/path/date.log for this.

Check the logs of container c3 to confirm correct setup.

---

 Volume should NOT be shared per POD
 
 ```
 k run multi-container-playground --image=nginx:1.17.6-alpine --dry-run=client -o yaml > q13.yaml
 k replace --force -f q13.yaml 
 k get pod
 k describe pod multi-container-playground 
 ```
 //path can be adjusted 
 //can use args also in addition to command
 
```
  - image: busybox
    name: c2
    command: ["bin/sh", "-c"]
    args:
    - while true; do
        date >> /vol/date.log;
        sleep 1;
      done
    volumeMounts:
    - mountPath: /vol
      name: volume
  - image: busybox
    name: c3
    command: ["bin/sh","-c"]
    args:
    - tail -f /vol/date.log
 ```
Logs
```
k logs multi-container-playground c1
k logs multi-container-playground c2
k logs multi-container-playground c3
```

# Q14

You're ask to find out following information about the cluster k8s-c1-H :

How many controlplane nodes are available?
How many worker nodes are available?
What is the Service CIDR?
Which Networking (or CNI Plugin) is configured and where is its config file?
Which suffix will static pods have that run on cluster1-node1?
Write your answers into file /opt/course/14/cluster-info, structured like this:

```
# /opt/course/14/cluster-info
1: [ANSWER]
2: [ANSWER]
3: [ANSWER]
4: [ANSWER]
5: [ANSWER]
```

---
 
```
/opt/course/14/cluster-info
q1:How many master nodes are available?
1
[k get node]

q2:How many worker nodes are available?
2
[k get node]

q3:What is the Pod CIDR of cluster1-worker1?
10.244.1.0/24
[k describe node | less -p PodCIDR]

q4:What is the Service CIDR?
10.96.0.0/12
[cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep range]
OR
[k get pod kube-apiserver-controlplane -n kube-system -o yaml | grep -i service]

q5:Which Networking (or CNI Plugin) is configured and where is its config file?
Weave, /etc/cni/net.d/10-weave.conflist
OR (in some cases)
Calico, /etc/cni/net.d/calico-kubeconfig, /etc/cni/net.d/10-canal.conflist
[find /etc/cni/net.d/]

q6:Which suffix will static pods have that run on cluster1-worker1?
-cluster1-worker1
[k get pod]
```
 
# Q15

Write a command into /opt/course/15/cluster_events.sh which shows the latest events in the whole cluster, ordered by time (metadata.creationTimestamp). Use kubectl for it.

Now delete the kube-proxy Pod running on node cluster2-node1 and write the events this caused into /opt/course/15/pod_kill.log.

Finally kill the containerd container of the kube-proxy Pod on node cluster2-node1 and write the events into /opt/course/15/container_kill.log.

Do you notice differences in the events both actions caused?


---

```
kubectl get events -A --sort-by=.metadata.creationTimestamp
echo "kubectl get events -A --sort-by=.metadata.creationTimestamp" > /opt/course/15/cluster_events.sh
bash /opt/course/15/cluster_events.sh

k get events -n kube-system > /opt/course/15/pod_kill.log

crictl ps | grep kube-proxy
crictl stop ab4ae2d9784d7

k get events -n kube-system > /opt/course/15/container_kill.log
```

 
 
 
# Q16

Write the names of all namespaced Kubernetes resources (like Pod, Secret, ConfigMap...) into /opt/course/16/resources.txt.

Find the project-* Namespace with the highest number of Roles defined in it and write its name and amount of Roles into /opt/course/16/crowded-namespace.txt.

---
Namespaced Resources

```
k create ns cka-master
k api-resources --namespaced -o name > /opt/course/16/resources.txt
OR
k api-resources --namespace=true | awk {'print $1'} > /opt/course/16/resources.txt

k get role -n <namespace> --no-headers | wc -l
```

# Q17

In Namespace project-tiger create a Pod named tigers-reunite of image httpd:2.4.41-alpine with labels pod=container and container=pod. Find out on which node the Pod is scheduled. Ssh into that node and find the containerd container belonging to that Pod.

Using command crictl:

Write the ID of the container and the info.runtimeType into /opt/course/17/pod-container.txt

Write the logs of the container into /opt/course/17/pod-container.log

---

Node -> Port -> Container 

```
k run tigers-reunite --image=httpd:2.4.41-alpine --labels "pod=container,container=pod" -n project-tiger
k get pod -n project-tiger  -o wide
 
crictl ps  | grep -i reunite 
//output into given file path

crictl logs 70f8623c3ad4d 
//output into given file path

k logs tigers-reunite -n project-tiger >> pod-container.log
//output into given file path
```


# Q18

There seems to be an issue with the kubelet not running on cluster3-node1. Fix it and confirm that cluster has node cluster3-node1 available in Ready state afterwards. You should be able to schedule a Pod on cluster3-node1 afterwards.

Write the reason of the issue into /opt/course/18/reason.txt.

---
Fix Kubelet Issue

```
ps aux | grep kubelet
service kubelet status
service kubelet start

systemctl status kubelet
systemctl start kubelet

journalctl -u kubelet.service -f
ps -ef | grep -i kubelet
whereis kubelet
[/usr/bin/kubelet]
//correct path in config file 
/etc/kubernetes/kubelet.conf

OR 

/etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

# Q19
NOTE: This task can only be solved if questions 18 or 20 have been successfully implemented and the k8s-c3-CCC cluster has a functioning worker node

Use context: kubectl config use-context k8s-c3-CCC

Do the following in a new Namespace secret. Create a Pod named secret-pod of image busybox:1.31.1 which should keep running for some time.

There is an existing Secret located at /opt/course/19/secret1.yaml, create it in the Namespace secret and mount it readonly into the Pod at /tmp/secret1.

Create a new Secret in Namespace secret called secret2 which should contain user=user1 and pass=1234. These entries should be available inside the Pod's container as environment variables APP_USER and APP_PASS.

Confirm everything is working.

---

Secret

```
k -n secret run secret-pod --image=busybox:1.31.1 --dry-run=client -o yaml -- sh -c "sleep 5d" > q19.yaml
OR
k -n secret run secret-pod --image=busybox:1.31.1 --dry-run=client -o yaml --command -- sleep 4800 > q19.yaml

k create -f /opt/course/19/secret1.yaml -n secret

k apply -f q19.yaml

k create secret generic secret2 --from-literal=user=user1 --from-literal=pass=1234 -n secret

k describe pod -n secret
```

# Q20

Your coworker said node cluster3-node2 is running an older Kubernetes version and is not even part of the cluster. Update Kubernetes on that node to the exact version that's running on cluster3-controlplane1. Then add this node to the cluster. Use kubeadm for this.
---

Version Upgrade

```
kubeadm version
kubectl version
kubelet --version

kubeadm upgrade node
apt-get update
apt-cache madison kublet | grep -i 1.24.1
apt-cache show kubectl | grep 1.24.1
apt-mark unhold kublet kubectl 
apt-get update && apt-get install kubectl=1.24.1-00 kubelet=1.24.1-00
apt-mark hold kublet kubectl

kubectl version --client
kubelet --version

sudo systemctl daemon-reload
sudo systemctl restart kubelet
service kubelet status

kubeadm token create --print-join-command
//use resulting command 'kubadmin join xyz:6443 --token txy etc.]
kubeadm token list
service kubelet status
```

# Q21
Create a Static Pod named my-static-pod in Namespace default on cluster3-controlplane1. It should be of image nginx:1.16-alpine and have resource requests for 10m CPU and 20Mi memory.

Then create a NodePort Service named static-pod-service which exposes that static Pod on port 80 and check if it has Endpoints and if it's reachable through the cluster3-controlplane1 internal IP address. You can connect to the internal node IPs from your main terminal.

---

Static Pod and Service 

```
cd /etc/kubernetes/manifests/

kubectl run my-static-pod --image=nginx:1.16-alpine --dry-run=client -o yaml > my-static-pod.yaml
//--requests "cpu=10m,memory=20Mi" 

k apply -f my-static-pod.yaml 

k get pod -A | grep my-static-pod

kubectl expose pod my-static-pod-cluster3-master1 --name static-pod-service --type=NodePort --port 80

k get svc,ep -l run=my-static-pod
```

# Q22
Check how long the kube-apiserver server certificate is valid on cluster2-controlplane1. Do this with openssl or cfssl. Write the exipiration date into /opt/course/22/expiration.

Also run the correct kubeadm command to list the expiration dates and confirm both methods show the same date.

Write the correct kubeadm command that would renew the apiserver server certificate into /opt/course/22/kubeadm-renew-certs.sh.

---

Certificate Validity

```
find /etc/kubernetes/pki | grep apiserver

openssl x509 -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep Validity -A2

kubeadm certs check-expiration | grep apiserver

kubeadm certs renew apiserver

kubectl -n kube-system get cm kubeadm-config -o yaml
```


# Q23
Node cluster2-node1 has been added to the cluster using kubeadm and TLS bootstrapping.

Find the "Issuer" and "Extended Key Usage" values of the cluster2-node1:

kubelet client certificate, the one used for outgoing connections to the kube-apiserver.
kubelet server certificate, the one used for incoming connections from the kube-apiserver.
Write the information into file /opt/course/23/certificate-info.txt.

Compare the "Issuer" and "Extended Key Usage" fields of both certificates and make sense of these.

---
Certificate Issue and Extended Key Usage

```
openssl x509 -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep Issuer
openssl x509 -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep "Extended Key Usage" -A1

openssl x509 -noout -text -in /var/lib/kubelet/pki/kubelet.crt | grep Issuer
openssl x509 -noout -text -in /var/lib/kubelet/pki/kubelet.crt | grep "Extended Key Usage" -A1
```

# Q24
There was a security incident where an intruder was able to access the whole cluster from a single hacked backend Pod.

To prevent this create a NetworkPolicy called np-backend in Namespace project-snake. It should allow the backend-* Pods only to:

connect to db1-* Pods on port 1111
connect to db2-* Pods on port 2222
Use the app label of Pods in your policy.

After implementation, connections from backend-* Pods to vault-* Pods on port 3333 should for example no longer work.

---

Network Policy

```
k apply -f q24.yaml
k get networkpolicies -n project-snake
```

# Q25
Make a backup of etcd running on cluster3-controlplane1 and save it on the controlplane node at /tmp/etcd-backup.db.

Then create any kind of Pod in the cluster.

Finally restore the backup, confirm the cluster is still working and that the created Pod is no longer with us.

---

ETCD Snapshot SAVE and RESTORE 
Run 'k get pods -A' before and after RESTORE operation

```
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db
cat /etc/kubernetes/manifests/etcd.yaml
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd

ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key

OR 

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /tmp/etcd-backup.db 

kubectl run test --image=nginx

ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db --data-dir /var/lib/etcd-backup

ETCDCTL_API=3 etcdctl --data-dir /var/lib/etcd-backup snapshot restore /tmp/etcd-backup.db 
//rm -r /var/lib/etcd-backup/*

cd /etc/kubernetes/manifests/
cat etcd.yaml | grep data-dir
//- --data-dir=/var/lib/etcd

vim /etc/kubernetes/manifests/etcd.yaml
//change etcd-data path

  - hostPath:
      path: /var/lib/etcd-backup
      type: DirectoryOrCreate
    name: etcd-data
    
journalctl -u kubelet.service | grep -i etcd   

```

 

