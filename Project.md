# DevOps Tooling Website Solution

## Introduction

This project implements a DevOps solution comprising the following components:

- **Infrastructure:** AWS
- **Web Server:** Red Hat Enterprise Linux 9
- **Database Server:** Ubuntu Linux with MySQL
- **Storage Server:** Red Hat Enterprise Linux 9 with NFS Server
- **Programming Language:** PHP
- **Code Repository:** GitHub

## Step 1 - Prepare the NFS Server

### 1. Launch an EC2 Instance with RHEL Operating System

![WhatsApp Image 2024-10-11 at 17 46 27_64a21f97](https://github.com/user-attachments/assets/37ae5a44-b364-4902-bb57-98de3d456b53)


### 2. Configure Logical Volume Management (LVM) on the Server

- Format the LVM as `xfs`.
- Create the following three logical volumes: `lv-opt`, `lv-apps`, and `lv-logs`.
- Mount these logical volumes to the `/mnt` directory as follows:
  - `lv-apps` on `/mnt/apps` – For use by the web servers
  - `lv-logs` on `/mnt/logs` – For storing web server logs
  - `lv-opt` on `/mnt/opt` – For Jenkins server use in a subsequent project

#### Create three 10GB volumes in the same availability zone as the NFS server EC2 instance, and attach all three volumes to the server.

![WhatsApp Image 2024-10-11 at 17 39 02_28e8bcd6](https://github.com/user-attachments/assets/b94b2b7e-fe83-4bce-b74c-ff92855d283e)


#### Open a terminal and begin configuration by connecting to the EC2 instance via SSH.

```bash
ssh -i "id_rsa.pem" ec2-user@ec2-44-211-197-20.compute-1.amazonaws.com
```
![SSH NFS](./images/ssh-nfs.png)

#### Use `lsblk` to verify attached block devices, located in the `/dev/` directory.

```bash
lsblk
```
![WhatsApp Image 2024-10-11 at 17 57 05_4af59649](https://github.com/user-attachments/assets/f0579dcc-8d29-4a25-a7e4-26bcf45c2b7e)


#### Use the `gdisk` utility to create a single partition on each disk.

```bash
sudo gdisk /dev/xvdb
```

```bash
sudo gdisk /dev/xvdc
sudo gdisk /dev/xvdd
```

![WhatsApp Image 2024-10-11 at 17 57 29_4f8daae8](https://github.com/user-attachments/assets/1a7d5867-6f8d-4d33-8e3d-3fd1f8bcfa21)


![WhatsApp Image 2024-10-11 at 17 57 58_f9f563fc](https://github.com/user-attachments/assets/a84e8b22-e4ae-4588-b078-b7200376ca2d)


![WhatsApp Image 2024-10-11 at 17 58 25_853db5f0](https://github.com/user-attachments/assets/6e37b52b-ecc6-45ce-b114-9d49b17e7fbf)


#### Verify partitions on each disk.

```bash
lsblk
```

![WhatsApp Image 2024-10-11 at 17 59 34_760234fc](https://github.com/user-attachments/assets/f60ca714-184f-4ad0-8fbc-0bbd16f60a38)


#### Install the `lvm` package.

```bash
sudo yum install lvm2 -y
```

![image](https://github.com/user-attachments/assets/3627d380-1b0e-46d6-b359-c2ca648a7a2c)


#### Create physical volumes (PVs) for LVM.

```bash
sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1
sudo pvs
```
![WhatsApp Image 2024-10-11 at 18 00 22_cb50d22b](https://github.com/user-attachments/assets/57469bed-46a4-4298-8fa9-c6facf002129)


#### Create a volume group (VG) named `webdata-vg` containing all three PVs.

```bash
sudo vgcreate webdata-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
sudo vgs
```
![WhatsApp Image 2024-10-11 at 18 03 35_cf3b8594](https://github.com/user-attachments/assets/8c92f87f-6f04-437f-ab0f-412921661b45)


#### Create three logical volumes within the VG.

```bash
sudo lvcreate -n lv-apps -L 9G webdata-vg
sudo lvcreate -n lv-logs -L 9G webdata-vg
sudo lvcreate -n lv-opt -L 9G webdata-vg

sudo lvs
```
![WhatsApp Image 2024-10-11 at 18 06 00_a52b86d1](https://github.com/user-attachments/assets/a4ce73db-a96b-443e-b2fb-2380785e4f64)


#### Verify the setup with `vgdisplay -v`.

```bash
sudo vgdisplay -v
lsblk
```

![WhatsApp Image 2024-10-11 at 18 14 48_ba6dd7e3](https://github.com/user-attachments/assets/3cabc143-64f3-4da3-8207-2ff15bc98ea0)


![WhatsApp Image 2024-10-11 at 18 17 37_37660e99](https://github.com/user-attachments/assets/e719d17b-0610-4513-8b63-3883d629010f)


#### Format the logical volumes with the `xfs` filesystem.

```bash
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
```
![Format Filesystem](./images/xfs.png)

#### Create and mount directories.

```bash
sudo mkdir /mnt/apps /mnt/logs /mnt/opt
sudo mount /dev/webdata-vg/lv-apps /mnt/apps
sudo mount /dev/webdata-vg/lv-logs /mnt/logs
sudo mount /dev/webdata-vg/lv-opt /mnt/opt
```
![WhatsApp Image 2024-10-11 at 18 44 11_f045ac1a](https://github.com/user-attachments/assets/dd27c81b-9ec4-40b5-a5db-7cb2bd5d290b)


### 3. Install and Start the NFS Server

```bash
sudo yum update -y
sudo yum install nfs-utils -y
```

![WhatsApp Image 2024-10-11 at 18 18 33_588b0ddf](https://github.com/user-attachments/assets/05a8828d-6632-4ccf-b562-8c24a9917632)

```bash
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
![WhatsApp Image 2024-10-12 at 01 19 14_23eef52d](https://github.com/user-attachments/assets/87513b07-71d9-47d9-a1ba-4a878d22ea43)

### 4. Export Mounts for Web Server Access

#### Set permissions and restart the NFS server.

```bash
sudo chown -R nobody: /mnt/apps /mnt/logs /mnt/opt
sudo chmod -R 777 /mnt/apps /mnt/logs /mnt/opt
sudo systemctl restart nfs-server.service
```
![Set Permissions](./images/permissions.png)

#### Configure access for clients within the same subnet.

```bash
sudo vi /etc/exports

/mnt/apps 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)

sudo exportfs -arv
```
![WhatsApp Image 2024-10-11 at 18 50 21_45b09f30](https://github.com/user-attachments/assets/b9f19e69-2761-41b8-8abf-c736944e1930)

![WhatsApp Image 2024-10-11 at 18 53 43_895e1093](https://github.com/user-attachments/assets/b94e7350-b87f-4e9b-aa02-072ae4a04fd6)

### 5. Check NFS Port and Configure Security Group

```bash
rpcinfo -p | grep nfs
```

![WhatsApp Image 2024-10-11 at 19 14 59_b92e9999](https://github.com/user-attachments/assets/52c416fa-29cf-41bd-8bae-3d43eb791afa)

