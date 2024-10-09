## Set up environment
- Authenticate with GCP and set project
```bash
gcloud auth login
```
```bash
gcloud config set project <YOUR_PROJECT_ID>
```
- Set the compute region and zone (Optional)
```bash
gcloud config set compute/region us-central1
```
```bash
gcloud config set compute/zone us-central1-a
```
## Create a GCE instance for the NFS server
```bash
gcloud compute instances create nfs-server \
    --zone=us-central1-a \
    --machine-type=e2-medium \
    --image-project=ubuntu-os-cloud \
    --image-family=ubuntu-2004-lts \
    --tags=nfs-server
```
## SSH into the GCE instance
```bash
gcloud compute ssh nfs-server
```
## Install and configure NFS server (ssh)
- Install NFS server on the VM
```bash
sudo apt update
```
```bash
sudo apt install nfs-kernel-server -y
```
- Create a directory for NFS sharing
```bash
sudo mkdir -p /var/nfs/shared
```
```bash
sudo chown nobody:nogroup /var/nfs/shared
```
```bash
sudo chmod 777 /var/nfs/shared
```
- Configure NFS exports
```bash
sudo nano /etc/exports
```
Copy paste the following line
```bash
/var/nfs/shared *(rw,sync,no_subtree_check,no_root_squash)
```
- Restart NFS service
```bash
sudo systemctl restart nfs-kernel-server
```
## Set up firewall rules (gcloud)
- Create a firewall rule
```bash
gcloud compute firewall-rules create allow-nfs \
    --action=ALLOW \
    --direction=INGRESS \
    --rules=tcp:2049,udp:2049 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=nfs-server
```
## Set up an NFS client to connect to the NFS server
- Create a client instance (optional if testing on GCE)
```bash
gcloud compute instances create nfs-client \
    --zone=us-central1-a \
    --machine-type=e2-medium \
    --image-project=ubuntu-os-cloud \
    --image-family=ubuntu-2004-lts
```
- SSH into the client machine
```bash
gcloud compute ssh nfs-client
```
-  Install NFS client tools
```bash
sudo apt update
```
```bash
sudo apt install nfs-common -y
```
- Mount the NFS share
```bash
sudo mkdir -p /mnt/nfs/shared
```
- Replace nfs-server with the external IP of your NFS server (you can find it using gcloud compute instances list) (optional, works well with the instance name)
```bash
sudo mount -t nfs nfs-server:/var/nfs/shared /mnt/nfs/shared
```
- Create files and dirs
```bash
/mnt/nfs/shared
```
## Make the NFS mount permanent (optional)
- To make the NFS mount persistent across reboots on the client, edit the /etc/fstab file on the client machine:
```bash
sudo nano /etc/fstab
```
Copy paste the following line
```bash
nfs-server:/var/nfs/shared /mnt/nfs/shared nfs defaults 0 0
```
## (Optional) Enable more secure access
- Publicly accessible NFS shares are not secure. If you want a secure configuration, consider restricting access to specific IP ranges in the firewall or using VPNs for private access.
```bash
gcloud compute firewall-rules update allow-nfs --source-ranges=YOUR_IP_RANGE
```
