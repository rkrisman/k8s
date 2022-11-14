# Install a K8s Cluster on Ubuntu 20.04 in 7 Steps

Kubernetes cluster install scripts with 1 control, 2 worker, and 1 NFS nodes
Includes Flannel, Multus, NGINX Ingress, K8s Dashboard, and KubeVirt

Installation steps:

1. Prepare 4 Ubuntu 20.04 servers: 1x control, 2x workers, 1x NFS server

2. Assign static IP addresses to all 4 servers as follows:
- control - 172.31.254.1/24
- nfs - 172.31.254.9/24
- worker1 - 172.31.254.11/24
- worker2 - 172.31.254.12/24

3. Install NFS server on NFS server node:
sudo su
wget https://raw.githubusercontent.com/rkrisman/k8s1c2wnfs/main/nfsserverinstall
chmod +x nfsserverinstall
./nfsserverinstall -h nfs -i 172.31.254.9 -p /data/nfs1 -n 172.31.254.0/24

4. Install K8s cluster on control node:
sudo su
wget https://raw.githubusercontent.com/rkrisman/k8s1c2wnfs/main/k8scontrolinstall
chmod +x k8scontrolinstall
./k8scontrolinstall -h control -i 172.31.254.1 -r 1.23.6 -n 172.30.0.0/16

5. Install K8s on worker nodes:
sudo su
wget https://raw.githubusercontent.com/rkrisman/k8s1c2wnfs/main/k8sworkerinstall
chmod +x k8sworkerinstall

./k8sworkerinstall -h worker1 -i 172.31.254.11 -r 1.23.6 -s 172.31.254.9

./k8sworkerinstall -h worker2 -i 172.31.254.12 -r 1.23.6 -s 172.31.254.9

6. Join K8s workers to cluster: Look for the output at the end of the K8s cluster node installation for an instruction to join the cluster like the following. Ensure to run the command using sudo.
Then you can join any number of worker nodes by running the following on each as root:

sudo kubeadm join 172.31.254.1:6443 --token xxxxx
--discovery-token-ca-cert-hash xxxxx

7. Install K8s addons on control node:
wget https://raw.githubusercontent.com/rkrisman/k8s1c2wnfs/main/k8saddonsinstall
chmod +x k8saddonsinstall
./k8saddonsinstall -s 172.31.254.9 -p /data/nfs1
