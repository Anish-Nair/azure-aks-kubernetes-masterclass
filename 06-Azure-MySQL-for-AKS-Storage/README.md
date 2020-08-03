# Use Azure Database for MySQL servers for AKS Workloads

## Step-01: Introduction
- What are the problems with MySQL Pod & Azure Disks? 
- How we are going to solve them using AWS RDS Database?

## Step-02: Create Azure Database for MySQL servers
- Go to Service **Azure Database for MySQL servers**
- Click on **Add**
- **Basics**
- **Project details**
  - Subscription: Free Trial
  - Resource Group: aks-rg1
- **Server Details**
  - Server name: akswebappdb
  - Data source: none
  - Location: (US) East US
  - Version: 5.7 (default)
  - **Compute + Storage**
    - Pricing Tier: Basic
    - VCore: 1
    - Storage: 5GB
    - Storage Auto Growth: Yes
    - Backup Retention: 7 days
    - Locally Redundant: Yes
- **Administrative Account**      
  - Admin username: dbadmin
  - Password: Redhat1449
  - Confirm password: Redhat1449
- **Review + Create**  


## Step-03: Create Kubernetes externalName service Manifest and Deploy
- Create mysql externalName Service
- **01-MySQL-externalName-Service.yml**
```yml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: ExternalName
  externalName: usermgmtdb.c7hldelt9xfp.us-east-1.rds.amazonaws.com
```
 - **Deploy Manifest**
```
kubectl apply -f kube-manifests/01-MySQL-externalName-Service.yml
```
## Step-04:  Connect to RDS Database using kubectl and create usermgmt schema/db
```
kubectl run -it --rm --image=mysql:5.7.22 --restart=Never mysql-client -- mysql -h usermgmtdb.c7hldelt9xfp.us-east-1.rds.amazonaws.com -u dbadmin -pdbpassword11

mysql> show schemas;
mysql> create database usermgmt;
mysql> show schemas;
mysql> exit
```
## Step-05: In User Management Microservice deployment file change username from `root` to `dbadmin`
- **02-UserManagementMicroservice-Deployment-Service.yml**
```yml
# Change From
          - name: DB_USERNAME
            value: "root"

# Change To
          - name: DB_USERNAME
            value: "dbadmin"            
```

## Step-06: Deploy User Management Microservice and Test
```
# Deploy all Manifests
kubectl apply -f kube-manifests/

# List Pods
kubectl get pods

# Stream pod logs to verify DB Connection is successful from SpringBoot Application
kubectl logs -f <pod-name>
```
## Step-07: Access Application
```
# Capture Worker Node External IP or Public IP
kubectl get nodes -o wide

# Access Application
http://<Worker-Node-Public-Ip>:31231/usermgmt/health-status
```

## Step-08: Clean Up 
```
# Delete all Objects created
kubectl delete -f kube-manifests/

# Verify current Kubernetes Objects
kubectl get all
```