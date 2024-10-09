## Set up the GKE cluster
- Create a GKE cluster
```bash
gcloud container clusters create nfs-test-cluster \
    --zone us-central1-a \
    --num-nodes 1
```
- Authenticate kubectl with the GKE cluster
```bash
gcloud container clusters get-credentials nfs-test-cluster --zone us-central1-a
```
## Create Kubernetes resources to use the NFS server
- Replace <NFS_SERVER_IP> with the external IP address of your NFS server (from the GCE instance).
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /var/nfs/shared
    server: <NFS_SERVER_IP> # REPLACE HERE
  storageClassName: nfs-storage-class

```
- Apply the StorageClass:
```bash
kubectl apply -f nfs-storage-class.yaml
```
- Apply the nfs-pv.yaml:
```bash
kubectl apply -f nfs-pv.yaml
```
- Apply the nfs-pvc.yaml:
```bash
kubectl apply -f nfs-pvc.yaml
```
- Apply the nfs-test-pod.yaml:
```bash
kubectl apply -f nfs-test-pod.yaml
```
## Verify the Setup
- Check if the PV is Bound:
```bash
kubectl get pv
```
- Check if the Pod is Running:
```bash
kubectl get pods
```
- Exec into the Pod and Verify the NFS Mount:
```bash
kubectl exec -it nfs-test-pod -- sh
```
- Run commands
```bash
cd /mnt/nfs
touch file-from-gke.txt
mkdir dir-from-gke
```
## Verify on the nsf-server side
- ssh to nfs server, cd to mounted directory and check for existance of created files/dirs
## Optionally run the nfs-setup-pod.yaml pod
- it will create tenantid dir with subdirectories for services (can be configured for multi tenant setup) so we can mount our services later
```bash
kubectl apply -f nfs-setup-pod.yaml
```
## Clean up (Optional)
```bash
gcloud container clusters delete nfs-test-cluster --zone us-central1-a
```
```bash
gcloud compute instances delete nfs-client --zone us-central1-a
```
```bash
gcloud compute instances delete nfs-server --zone us-central1-a
```
## Or resize cluster for further use
```bash
gcloud container clusters resize nfs-test-cluster --size=0
```
