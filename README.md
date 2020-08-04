# gke-with-day-two-operations

## Day 2 K8s (on GKE infra) with Ansible
  
**"quickly configure/provision cluster-level components and workloads"** 

-------------------------

#### **parent repo:** https://github.com/consistent-solutions/terra-gke-jumpstart  
**"spin up prerequisite GKE Public Cluster via Terraform"**    

--------------------------

## Cluster level Features
### __2048 Game__
##### Integrates with Cert-Manager for Ingress-based TLS termination
##### Deployment, NodePort Service and Ingress **+** Google External HTTP(S) LoadBalancer   
https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer   
### __ExternalDNS__
##### "keep zones synchronized with Ingresses and Services (of type=LoadBalancer) in various cloud providers."   
https://github.com/kubernetes-sigs/external-dns#the-latest-release-v07   
### __Cert-Manager__
##### "x509 certificate management for Kubernetes" for Ingress resources via Let's Encrypt (with DNS01 Challenge).   
https://github.com/jetstack/cert-manager      
### __Velero__
##### Disaster Recovery. With supported backup scenarios:      
- Entire Cluster   
- By Label   
- By Namespace   
- Via Cron Schedule   
- Workload Migration   

https://github.com/vmware-tanzu/velero

--------------------------

### 1) Provision GKE Public Cluster
- complete all steps prior to https://github.com/consistent-solutions/terra-gke-jumpstart#configure-k8s-rbac-and-provision-workloads-with-ansible.
#### NOTE: for simplicity use Google Cloud Shell for following steps.
### 2) Install Ansible

### Ensure Ansible is version 2.8 or greater (GCP Ansible modules require Ansible version 2.8 or newer)
#### NOTE: may want to run with ```--force-reinstall``` option to avoid installing side-by-side with other installed versions   
- ```pip install ansible==2.9.6```

### 3) Create Ansible OWNER Service Account
#### NOTE: (for ansible to run with gcp modules) an OWNER service account was MANUALLY created with:
- ```gcloud iam service-accounts create ansible --display-name=ansible```
- ```gcloud iam projects add-iam-policy-binding gke-day-two --member serviceAccount:ansible@gke-day-two.iam.gserviceaccount.com --role roles/owner```

### 4) Run Ansible Playbook
- ```cd gke-with-day-two-operations/ansible```
- ```~/.local/bin/ansible-playbook -i environments/dev dev.yml```

#### NOTE: IT can take 7-10 minutes for the game to be accessible!!

### 5) Teardown Ansible Workloads:
- ```~/.local/bin/ansible-playbook -i environments/dev delete_k8s_cluster_app_resources.yml```

--------------------------

## Velero Backup Scenarios
#### NOTE:  Verify default backup provider and location with ```velero backup-location get```
#### NOTE: List backups/restores/scheduled backups with ```velero <backup/restore/schedule> get```
#### NOTE: Use ```--snapshot-volumes``` to include backup of underlying disks for PVs.
#### NOTE: Use ```--include-namespaces=<desired-namespace>``` option with restore or create for specific namespace.

### 1. Entire cluster:
NOTE: velero will not restore resources if they already exist
```
# create velero backup
velero backup create full-cluster-backup
# get progress and details for backup
velero backup describe full-cluster-backup
#(once Phase/Status: InProgress ---> Phase/Status: Complete)
# simulate disaster
kubectl delete ns test-mysql
# confirm disaster
kubectl get po -n test-mysql
# restore from backup
velero restore create --from-backup full-cluster-backup
# get progress and details for restore
velero restore describe full-cluster-backup
# (once Phase is Complete)
kubectl get po -n test-mysql
# remove backup
velero backup delete full-cluster-backup
```

#### 2. By label: ####
```
# create velero backup
velero backup create test-selector-backup --selector app=wordpress
# check progress/status of backup
velero backup describe test-selector-backup
# (once Phase/Status: InProgress ---> Phase/Status: Complete)
# simulate disaster
kubectl delete ns test-mysql
# confirm disaster
kubectl get po -n test-mysql
# restore from backup
velero restore create --from-backup test-selector-backup
# get progress and details for restore
velero restore describe test-selector-backup
# (once Phase is Complete)
kubectl get po -n test-mysql
# remove backup
velero backup delete test-selector-backup
```

#### 3. By namespace: ####
```
# create velero backup
velero backup create test-ns-backup --include-namespaces=test-mysql
# check progress/status of backup
velero backup describe test-ns-backup
# (once Phase/Status: InProgress ---> Phase/Status: Complete)
# simulate disaster
kubectl delete ns test-mysql
# confirm disaster
kubectl get po -n test-mysql
# restore from backup
velero restore create --from-backup test-ns-backup
# get progress and details for restore
velero restore describe test-ns-backup
# (once Phase is Complete)
kubectl get po -n test-mysql
# remove backup
velero backup delete test-ns-backup
```

#### 4. Via cron schedule: ####
```
# create scheduled backup (every morning at 7am) (kicks off first backup immediately)
velero create schedule test-schedule-backup --schedule "00 7 * * *"
# list scheduled backups
velero schedule get
# list backups
velero backup get
# (once Phase/Status: InProgress ---> Phase/Status: Complete)
# simulate disaster
kubectl delete ns test-mysql
# confirm disaster
kubectl get po -ns test-mysql
# get backups (for <#######> on next step)
velero backup get
# restore from schedule backup
velero restore create --from-backup test-schedule-backup-<######>
# check status of restore
velero restore get test-schedule-backup-<######>
# (once Phase is Complete)
# confirm restore
kubectl get po -n test-mysql
# delete scheduled backup
velero schedule delete test-schedule-backup
# delete backup
velero backup delete test-schedule-backup-<######>
```
