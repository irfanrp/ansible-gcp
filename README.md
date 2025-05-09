# Kubernetes Cluster Provisioning with Ansible

This project provisions a Kubernetes cluster (1 master + 2 workers) using **Ansible** ,  **kubectl**, **kubeadm** &  **kubelet**.

No minikube, no k3s → full kubeadm cluster.

---

## 📂 **Deliverables**

This repository contains:

- `playbooks/` → Ansible playbooks
- `inventory/` → Ansible inventory file
- `templates/` → kubeadm config template

As required in deliverables:
> “Ansible code (playbooks, inventory files, vars)”

---

## 📝 **Prerequisites**

- Ubuntu 20.04+ servers (tested on GCP)
- SSH access with Ansible control node
- Ansible ≥ 2.18.4 installed on control machine
- Internet access for nodes

## 🔥 Firewall/Network Requirements

Agar cluster Kubernetes berjalan normal, pastikan port berikut terbuka:

| Port               | Direction | Node Target   | Kegunaan                                 |
|-------------------|-----------|---------------|------------------------------------------|
| 22/tcp             | ingress   | Semua node    | SSH access                               |
| 6443/tcp           | ingress   | Master        | Kubernetes API server                    |
| 2379-2380/tcp      | ingress   | Master        | etcd server client API                   |
| 10250/tcp          | ingress   | Semua node    | Kubelet API                              |
| 10251/tcp          | ingress   | Master        | kube-scheduler                           |
| 10252/tcp          | ingress   | Master        | kube-controller-manager                  |
| 30000-32767/tcp    | ingress   | Semua node    | NodePort services (akses app via NodePort) |

> ✅ **GCP firewall commands:**

```bash
gcloud compute firewall-rules create kubernetes-allow-ssh \
  --allow tcp:22 --target-tags=kubernetes --description="Allow SSH" --direction=INGRESS

gcloud compute firewall-rules create kubernetes-allow-api-server \
  --allow tcp:6443 --target-tags=kubernetes --description="Allow Kubernetes API server" --direction=INGRESS

gcloud compute firewall-rules create kubernetes-allow-etcd \
  --allow tcp:2379-2380 --target-tags=kubernetes --description="Allow etcd ports" --direction=INGRESS

gcloud compute firewall-rules create kubernetes-allow-kubelet \
  --allow tcp:10250 --target-tags=kubernetes --description="Allow kubelet API" --direction=INGRESS

gcloud compute firewall-rules create kubernetes-allow-scheduler \
  --allow tcp:10251 --target-tags=kubernetes --description="Allow kube-scheduler" --direction=INGRESS

gcloud compute firewall-rules create kubernetes-allow-controller-manager \
  --allow tcp:10252 --target-tags=kubernetes --description="Allow kube-controller-manager" --direction=INGRESS

gcloud compute firewall-rules create kubernetes-allow-nodeport \
  --allow tcp:30000-32767 --target-tags=kubernetes --description="Allow NodePort range" --direction=INGRESS
---

## ⚙️ **How to deploy**

Run the playbooks in order:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/install-prerequisites.yml
ansible-playbook -i inventory/hosts.ini playbooks/init-master.yml
ansible-playbook -i inventory/hosts.ini playbooks/join-workers.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy-nginx.yml

