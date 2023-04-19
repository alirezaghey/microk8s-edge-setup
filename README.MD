# MicroK8s HA Cluster on the Edge

## Synopsis
The goal is to setup an MicroK8s HA cluster on the edge. The setup is as follows:
1. 1 * Teltonika RUTX08 router with wireguard VPN.
2. 3+ * RasberiPi 4 with Ubuntu Server (Free/Pro/Core)


### How to Create and Clone the Clean Ubuntu Images
[./rpi-images/README.MD](./rpi-images/README.MD)


### Ansible Playbooks to Init and Set Up Rpis with Microk8s
[./ansible/README.MD](./ansible/README.MD)