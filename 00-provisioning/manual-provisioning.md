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
```

<hr>


## 4. Provision a User Managed Service Account (UMSA)

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


## 5. Grant requisite permissions to the UMSA and yourself

### 5.1. Grant the UMSA requisite permissions

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

```

### 5.2. Grant yourself permissions to impersonate the UMSA

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


## 6. Provision the Networking Dependencies

### 6.1. Create a VPC

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
VPC_NM="ray-lab-vpc"
UMSA=lab-sa
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com

gcloud compute networks create $VPC_NM \
--project=$PROJECT_ID \
--subnet-mode=custom \
--mtu=1460 \
--bgp-routing-mode=regional \
--impersonate-service-account $UMSA_FQN
```

### 6.2. Create a subnet

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
VPC_NM="ray-lab-vpc"
SUBNET_NM="ray-lab-snet"
RAY_LAB_SUBNET_CIDR=10.0.0.0/16
UMSA=lab-sa
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
LOCATION="us-central1"


gcloud compute networks subnets create $SUBNET_NM \
 --network $VPC_NM \
 --range $RAY_LAB_SUBNET_CIDR  \
 --region $LOCATION \
 --enable-private-ip-google-access \
 --project $PROJECT_ID \
--impersonate-service-account $UMSA_FQN
```

### 6.3. Create peering with service networking

Paste in Cloud Shell scoped to the project you created-
```
PROJECT_ID=`gcloud config list --format "value(core.project)" 2>/dev/null`
VPC_NM="ray-lab-vpc"
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

## 7. Provision Cloud Storage Buckets

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

## 8. Provision a BigQuery Dataset

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

## 9. Provision Ray on Vertex AI 

We want t 

## 11. Upload lab assets

### 11.1. Upload notebooks

### 11.2. Upload data



