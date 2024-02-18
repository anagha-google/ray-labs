# Manual Provisioning of the lab environment on GCP


## 1. Create a GCP project

Create a project manually

<hr>

## 2. Grant yourself requisite privileges to set up the environment in your project

This includes Owner (basic) permission and Organization policy administrator role (applicable for GCP CE Argolis environment).

<hr>

## 3. Enable requisite Google APIs

Paste in Cloud Shell scoped to the project you created-
```
gcloud services enable orgpolicy.googleapis.com
gcloud services enable compute.googleapis.com
gcloud services enable servicenetworking.googleapis.com
gcloud services enable container.googleapis.com
gcloud services enable artifactregistry.googleapis.com
gcloud services enable containerregistry.googleapis.com
gcloud services enable bigquery.googleapis.com 
gcloud services enable storage.googleapis.com
gcloud services enable aiplatform.googleapis.com
gcloud services enable notebooks.googleapis.com
gcloud services enable logging.googleapis.com
gcloud services enable monitoring.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable dataproc.googleapis.com
gcloud services enable dataplex.googleapis.com
gcloud services enable datacatalog.googleapis.com
gcloud services enable datalineage.googleapis.com
gcloud services enable dataform.googleapis.com
```

<hr>

## 4. Update Organization Policies
This is for any compute you may spin up for ETL or notebooks...such as Dataproc for Spark-based ETL, Notebook instances on Vertex AI Workbench managed instances.<br>

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
PROJECT_NBR=`gcloud projects describe $PROJECT_ID | grep projectNumber | cut -d':' -f2 |  tr -d "'" | xargs`
PROJECT_NAME=`gcloud projects describe ${PROJECT_ID} | grep name | cut -d':' -f2 | xargs`

#4.a. Relax require OS Login

rm -rf os_login.yaml

cat > os_login.yaml << ENDOFFILE
name: projects/${PROJECT_ID}/policies/compute.requireOsLogin
spec:
  rules:
  - enforce: false
ENDOFFILE

gcloud org-policies set-policy os_login.yaml 

rm -rf os_login.yaml

#-------------------------------------

#4.b. Disable Serial Port Logging

rm -rf disableSerialPortLogging.yaml

cat > disableSerialPortLogging.yaml << ENDOFFILE
name: projects/${PROJECT_ID}/policies/compute.disableSerialPortLogging
spec:
  rules:
  - enforce: false
ENDOFFILE

gcloud org-policies set-policy disableSerialPortLogging.yaml 

rm -rf disableSerialPortLogging.yaml

#-------------------------------------

#4.c. Disable Shielded VM requirement

rm -rf shieldedVm.yaml 

cat > shieldedVm.yaml << ENDOFFILE
name: projects/$PROJECT_ID/policies/compute.requireShieldedVm
spec:
  rules:
  - enforce: false
ENDOFFILE

gcloud org-policies set-policy shieldedVm.yaml 

rm -rf shieldedVm.yaml

#-------------------------------------

#4.d. Disable VM can IP forward requirement

rm -rf vmCanIpForward.yaml

cat > vmCanIpForward.yaml << ENDOFFILE
name: projects/$PROJECT_ID/policies/compute.vmCanIpForward
spec:
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy vmCanIpForward.yaml

rm -rf vmCanIpForward.yaml

#-------------------------------------

#4.e. Enable VM external access

rm -rf vmExternalIpAccess.yaml

cat > vmExternalIpAccess.yaml << ENDOFFILE
name: projects/$PROJECT_ID/policies/compute.vmExternalIpAccess
spec:
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy vmExternalIpAccess.yaml

rm -rf vmExternalIpAccess.yaml

#-------------------------------------

#4.f. Enable restrict VPC peering

rm -rf restrictVpcPeering.yaml

cat > restrictVpcPeering.yaml << ENDOFFILE
name: projects/$PROJECT_ID/policies/compute.restrictVpcPeering
spec:
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy restrictVpcPeering.yaml

rm -rf restrictVpcPeering.yaml

#-------------------------------------

```

<hr>


## 5. Provision a User Managed Service Account (UMSA)

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
PROJECT_NBR=`gcloud projects describe $PROJECT_ID | grep projectNumber | cut -d':' -f2 |  tr -d "'" | xargs`
PROJECT_NAME=`gcloud projects describe ${PROJECT_ID} | grep name | cut -d':' -f2 | xargs`
UMSA=lab-sa


gcloud iam service-accounts create ${UMSA} \
    --description="User Managed Service Account for the Ray lab" \
    --display-name=$UMSA 

```

<hr>

## 6. Grant requisite permissions to the UMSA and yourself

### 6.1. Grant the UMSA requisite permissions

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
PROJECT_NBR=`gcloud projects describe $PROJECT_ID | grep projectNumber | cut -d':' -f2 |  tr -d "'" | xargs`
PROJECT_NAME=`gcloud projects describe ${PROJECT_ID} | grep name | cut -d':' -f2 | xargs`
UMSA=lab-sa
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member=serviceAccount:${UMSA_FQN} \
    --role=roles/iam.serviceAccountUser
    
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member=serviceAccount:${UMSA_FQN} \
    --role=roles/iam.serviceAccountTokenCreator 
    
gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/bigquery.dataEditor"

gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/bigquery.jobUser"

gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/bigquery.admin"

gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/storage.admin"

gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/storage.objectCreator"

gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/aiplatform.admin"

gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/compute.networkAdmin"

gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/dataplex.admin"

gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/datalineage.admin"

gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN \
--role="roles/datacatalog.admin"

```

### 6.2. Grant yourself permissions to impersonate the UMSA

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
PROJECT_NBR=`gcloud projects describe $PROJECT_ID | grep projectNumber | cut -d':' -f2 |  tr -d "'" | xargs`
PROJECT_NAME=`gcloud projects describe ${PROJECT_ID} | grep name | cut -d':' -f2 | xargs`
UMSA=lab-sa
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
YOUR_GCP_ACCOUNT_NAME=`gcloud auth list --filter=status:ACTIVE --format="value(account)"`

gcloud iam service-accounts add-iam-policy-binding \
    ${UMSA_FQN} \
    --member="user:${YOUR_GCP_ACCOUNT_NAME}" \
    --role="roles/iam.serviceAccountUser"
    
gcloud iam service-accounts add-iam-policy-binding \
    ${UMSA_FQN} \
    --member="user:${YOUR_GCP_ACCOUNT_NAME}" \
    --role="roles/iam.serviceAccountTokenCreator"
```

<hr>

## 7. Provision the Networking Dependencies

Ray (on Vertex AI) clusters are created in the Google tenant project and not your GCP project. And they are private by default. So to connect with them, from your compute (Vertex AI workbench instance), you need to peer your compute network with Google Service Networking. This section addresses creation of all requisite networking functions.

### 7.1. Create a VPC

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
VPC_NM="ray-lab-vpc-auto-snet"
UMSA=lab-sa
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com

gcloud compute networks create $VPC_NM \
--project=$PROJECT_ID \
--subnet-mode=auto \
--mtu=1460 \
--bgp-routing-mode=regional \
--impersonate-service-account $UMSA_FQN
```

### 7.3. Create peering with service networking

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
VPC_NM="ray-lab-vpc-auto-snet"
UMSA=lab-sa
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
PEERING_NM="ray-lab-peering-to-service-networking"

# This is for display only; you can name the range anything.
PEERING_RANGE_NAME=ray-lab-vpc-peering-reserved-range

# Create the reserved range
# NOTE: `prefix-length=16` means a CIDR block with mask /16 will be
# reserved for use by Google services, such as Vertex AI.

gcloud compute addresses create $PEERING_RANGE_NAME \
  --global \
  --prefix-length=16 \
  --description="Peering range for Google service" \
  --network=$VPC_NM \
  --purpose=VPC_PEERING \
  --impersonate-service-account $UMSA_FQN

# Create the VPC peering
gcloud services vpc-peerings connect \
  --service=servicenetworking.googleapis.com \
  --network=$VPC_NM \
  --ranges=$PEERING_RANGE_NAME \
  --project=$PROJECT_ID \
  --impersonate-service-account $UMSA_FQN
```
<hr>

## 8. Provision Cloud Storage Buckets

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
PROJECT_NBR=`gcloud projects describe $PROJECT_ID | grep projectNumber | cut -d':' -f2 |  tr -d "'" | xargs`
UMSA=lab-sa
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
LOCATION="us-central1"
LAB_DATA_BUCKET=ray_lab_data_bucket_$PROJECT_NBR
LAB_CODE_BUCKET=ray_lab_code_bucket_$PROJECT_NBR


gcloud storage buckets create gs://$LAB_DATA_BUCKET --location=$LOCATION --impersonate-service-account $UMSA_FQN
gcloud storage buckets create gs://$LAB_CODE_BUCKET --location=$LOCATION --impersonate-service-account $UMSA_FQN
```

<hr>

## 9. Provision a BigQuery Dataset

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
PROJECT_NBR=`gcloud projects describe $PROJECT_ID | grep projectNumber | cut -d':' -f2 |  tr -d "'" | xargs`
UMSA=lab-sa
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
LOCATION="us-central1"
BQ_DATASET_NM="ray_lab_ds"

bq --location=$LOCATION mk \
    --dataset \
    --service_account $UMSA_FQN \
    $PROJECT_ID:$BQ_DATASET_NM 
```

<hr>
<hr>

## 10. Create a Ray on Vertex cluster manually

### 10.1. Background:
1. Currently, there is no gcloud support for creating Ray on Vertex AI
2. There is SDK support for creating Ray clusters on Vertex
3. For the purpose of simplicity, we will create a cluster manually, via the user interface

<hr>

### 10.2. Create a cluster

This takes about 15 minutes. Follow the steps below-


![LAB](images/m00-lab-01.png)   
<br><br>

![LAB](images/m00-lab-02.png)   
<br><br>

![LAB](images/m00-lab-03.png)   
<br><br>

![LAB](images/m00-lab-04.png)   
<br><br>

![LAB](images/m00-lab-05.png)   
<br><br>

![LAB](images/m00-lab-06.png)   
<br><br>

![LAB](images/m00-lab-07.png)   
<br><br>

<hr>

### 10.3. Walk-through of the cluster and UIs and terminals

Lets familiarize ourselves with the cluster -

#### 10.3.1. The cluster

![LAB](images/m00-lab-08.png)   
<br><br>

![LAB](images/m00-lab-09.png)   
<br><br>

![LAB](images/m00-lab-10.png)   
<br><br>

![LAB](images/m00-lab-11.png)   
<br><br>

#### 10.3.2. The Ray UI

![LAB](images/m00-lab-12.png)   
<br><br>

![LAB](images/m00-lab-13.png)   
<br><br>

#### 10.3.3. The terminal to the head node 

![LAB](images/m00-lab-14.png)   
<br><br>

![LAB](images/m00-lab-15.png)   
<br><br>


<hr>

## 11. Review of Colab Enterprise and RoV integration

Ray on Vertex cluster creation results in auto-creation of a Colab Enterprise Runtime Template with Ray libraries (CERTFROV) pre-installed and compatible with the RoV cluster. Launching a notebook that uses the CERTFROV provides a smooth experience working with Ray via notebooks, without any Ray specific compatibility issues across Colab and RoV cluster.

### 11.1. Getting to the Colab Enterprise from the RoV cluster 

![LAB](images/m00-lab-16.png)   
<br><br>

### 11.2. The notebook auto-created by the RoV cluster creation process for you

![LAB](images/m00-lab-17.png)   
<br><br>

### 11.2. The Colab Runtime Template for RoV

This runtime template gets automatically created for you so that you can semaless connect to the RoV cluster from Colab without having to install Ray and fix any version incompatibilities.


![LAB](images/m00-lab-18.png)   
<br><br>

![LAB](images/m00-lab-19.png)   
<br><br>

### 11.3. Connecting the auto-created notebook to the auto-created runtime template

Follow the steps below to get started-

![LAB](images/m00-lab-20.png)   
<br><br>

![LAB](images/m00-lab-21.png)   
<br><br>

![LAB](images/m00-lab-22.png)   
<br><br>

![LAB](images/m00-lab-23.png)   
<br><br>

### 11.4. Smoke test the cluster created

Follow the steps below to get started-


1. Check to see if you are connected to rhe Ray cluster
   
![LAB](images/m00-lab-24.png)   
<br><br>

2. Execute the first two cells

  <hr>

3. Open the Ray dashboard at the link in the notebook

![LAB](images/m00-lab-25.png)   
<br><br>

4. Switch back to the noteook and paste the below into a cell, and run this "Hello World" program to familiarize yourself with remote job execution-

```
import time

@ray.remote
def hello_world():
    return "hello world"

@ray.remote
def square(x):
    print(x)
    time.sleep(100)
    return x * x

ray.init()  # No need to specify address="vertex_ray://...."
print(ray.get(hello_world.remote()))
print(ray.get([square.remote(i) for i in range(4)]))
```


5. You should see the execution in the cell output as shown below

![LAB](images/m00-lab-26.png)   
<br><br>

6. Switch to the dashboard & study the various tabs

![LAB](images/m00-lab-27.png)   
<br><br> 


![LAB](images/m00-lab-28.png)   
<br><br> 



![LAB](images/m00-lab-29.png)   
<br><br> 


![LAB](images/m00-lab-30.png)   
<br><br> 


<hr>

## 12. Load data into BigQuery
### 12.1. Download the dataset

Paste in Cloud Shell scoped to your project-
```
cd ~
mkdir ray-downloads
cd ray-downloads

wget https://github.com/anagha-microsoft/ncr-mlops-hol/blob/master/data/framingham.csv .

```

### 12.2. Load data into BigQuery

```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
PROJECT_NBR=`gcloud projects describe $PROJECT_ID | grep projectNumber | cut -d':' -f2 |  tr -d "'" | xargs`
LOCATION="us-central1"
BQ_DATASET_NM="ray_lab_ds"

cd ~/ray-downloads

bq load --autodetect=true --source_format="CSV" --replace=true "$BQ_DATASET_NM.chd_source" framingham.csv

<hr>

## 13. Upload notebooks into Colab Enterprise



<hr><hr>

This concludes the module, proceed to the next module.

<hr>

