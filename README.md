# Installing K8s Cluster on Ubuntu 20.04 in 8 Easy Steps

Kubernetes cluster install scripts with 1 control, 2 worker, and 1 NFS nodes
Includes Flannel, Multus, NGINX Ingress, K8s Dashboard, and KubeVirt

Installation steps:

1. Prepare 4 Ubuntu 20.04 servers: 1x control, 2x workers, 1x NFS server

2. Assign static IP addresses to all 4 servers, such as the following:
- control - 172.31.254.1/24
- nfs - 172.31.254.9/24
- worker1 - 172.31.254.11/24
- worker2 - 172.31.254.12/24

3. Install NFS server on the NFS server node:

echo -e "<password>" | sudo -S su

wget https://raw.githubusercontent.com/rkrisman/k8s1c2wnfs/main/nfsserverinstall

chmod +x nfsserverinstall

./nfsserverinstall -h nfs -i <nfs-node-ip> -p <nfs-share> -n <allowed-cidr>

Example:
./nfsserverinstall -h nfs -i 172.31.254.9 -p /data/nfs1 -n 172.31.254.0/24

4. Install K8s on control node:

echo -e "<password>" | sudo -S su

wget https://raw.githubusercontent.com/rkrisman/k8s1c2wnfs/main/k8scontrolinstall

chmod +x k8scontrolinstall

Run the K8s control node installation script with the following format:
./k8scontrolinstall -h control -i <control-node-ip> -r <k8s-release> -n <node-cidr>

Example:
./k8scontrolinstall -h control -i 172.31.254.1 -r 1.25.3 -n 172.30.0.0/16

5. Install K8s on worker nodes:

echo -e "<password>" | sudo -S su

wget https://raw.githubusercontent.com/rkrisman/k8s1c2wnfs/main/k8sworkerinstall

chmod +x k8sworkerinstall

Run the K8s worker node installation script with the following format:
./k8scontrolinstall -h control -i <worker-node-ip> -r <k8s-release> -s <nfs-server-ip>

worker1 example:
./k8sworkerinstall -h worker1 -i 172.31.254.11 -r 1.25.3 -s 172.31.254.9

worker2 example:
./k8sworkerinstall -h worker2 -i 172.31.254.12 -r 1.25.3 -s 172.31.254.9

6. Join K8s workers to cluster: Look for the output at the end of the K8s cluster node installation for an instruction to join the cluster like the following. Ensure to run the command using sudo. Then you can join any number of worker nodes by running the following on each as root:

sudo kubeadm join <control-node-ip>:6443 --token xxxxx --discovery-token-ca-cert-hash xxxxx

Example:
sudo kubeadm join 172.31.254.1:6443 --token e9jsaq.m6ctbxe0gznirlpf \
        --discovery-token-ca-cert-hash sha256:754ea651bbb2cdd0a5a3639d700a4af0f3418f8f3c1804156fd0573b9338eedc

7. Install K8s addons (NFS Provisioner, Ingress NGINX, KubeVirt, CDI, Dashboard) on control node:

echo -e "fortinet" | sudo -S su

wget https://raw.githubusercontent.com/rkrisman/k8s1c2wnfs/main/k8saddonsinstall

chmod +x k8saddonsinstall

./k8saddonsinstall -s <nfs-node-ip> -p <nfs-share>

Example:
./k8saddonsinstall -s 172.31.254.9 -p /data/nfs1

8. Configure static host mapping for the K8s Dashboard:

edit /etc/hosts in Linux or C:\Windows\System32\drivers\etc\hosts in Windows to include the following entry

<control-node-ip>  dashboard.k8s.lab

Example:
172.31.254.1  dashboard.k8s.lab

Then, access the K8s Dashboard by browsing to https://dashboard.k8s.lab:30443
