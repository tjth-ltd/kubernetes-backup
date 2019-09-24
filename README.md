# kubernetes-backup

Kubernetes Backup Project (Based on Volero and Minio for Object Storage).

This project contains two directories for the two separate components for the Kubernetes backup configuration. Configuration and installation instructions can be found below.

More information: https://tjth.co/kubernetes-backups-with-velero/

## backup-storage (MinIO)

This docker-compose container should be setup on the intended destination for the Kubernetes Backups. The docker-compose file will start a single minIO instance, mounting a directory in which it will share "buckets" which function in the same way as AWS S3 buckets do.

### Configuration

Before deploying the minIO destination, you should update the docker-compose.yml file with the directory in which the "buckets" should be created.

### Installation

* Clone this repository to your backup destination
* Update the docker-compose with the intended directory for the buckets
* Start the container:
```
sudo docker-compose up -d
```
* When the container has started, execute the following to get the ACCESS KEY & SECRET KEY:
```
sudo docker exec -it kubernetes-backup-storage cat /data/.minio.sys/config/config.json | egrep "(access|secret)Key"
```
* Next, login to the web UI with credentials above for Minio and create a bucket with the name of the kubernetes cluster you intend to backup, eg "k8s-dev". This will be used later in the Velero configuration.
* Finally, make a note of the ACCESS KEY & SECRET KEY as the config file for the Velero deployment will need to be updated with these before it is installed.

## kubernetes-deployment (Velero)

### Configuration

Before running the Ansible playbook to deploy the Velero kubernetes package, update the Playbook with the DNS name for the backup server running minIO. This will be used to install Velero. 

### Installation

Run the ansible playbook to deploy the Velero installation to your cluster (Ensure to set hosts option and variables for the minio server before running).

### Usage

To run a once-off full backup of the cluster to your minio bucket, run the below:
```
velero backup create full-backup
```

Backups can be scheduled using crontab on the k8s-master (Or executed via ssh from another server if required / no master access available).

### Uninstall

To uninstall velero in the event of something going wrong, run the following commands:
```
kubectl delete namespace/velero clusterrolebinding/velero
kubectl delete crds -l component=velero
```