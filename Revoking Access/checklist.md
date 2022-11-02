
# Check List. Revoking Access to Google Cloud Platform

## According to officail [Google guidlance](https://cloud.google.com/docs/security/data-loss-prevention/revoking-user-access#revoking_access)

### **1. Remove the account from project membership**

```bash=1
# Find all **IAM policies** granted to the specific account
gcloud asset search-all-iam-policies --scope=<scope-area>/ID --query="policy:fired_user@exapmle.com" --sort-by=resource --format="table(assetType, resource, policy.bindings.role[])"

# Remove **IAM policy** granted to the specific principal
gcloud resource-manager <RESOURCE_TYPE> remove-iam-policy-binding 355580441671  --member=user:fired_user@example.com --role=roles/<fired_user_role>
```

### **2. Rotate project credentials**

* Rotate Service account keys. ***If the person whose access is being revoked was not an owner, you may skip this step***

```bash=1
#Find all **service account keys** used for specific projects
gcloud asset search-all-resources --scope='projects/rp-newtest' --asset-types='iam.googleapis.com/ServiceAccountKey'

#List public keys data for a service account key
gcloud iam service-accounts keys list  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

#Disable a service account key
gcloud iam service-accounts keys disable KEY_ID --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

#Create a service account key
#KEY_FILE: The path to a new output file for the private key—for example, ~/sa-private-key.json
#Make sure to store the key file securely, because it can be used to authenticate as your service account. You can move and rename this file however you would like.
#You can use service account key files to authenticate an application as a service account.

gcloud iam service-accounts keys create KEY_FILE --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com 
```

* Reset OAuth 2.0 client ID secrets. ***If the person whose access is being revoked was not an owner or editor, you may skip this step***

>there's no CLI command to reset OAuth client secret, so use [credentials interface](https://console.cloud.google.com/apis/credentials/) to reset it manual
>
> *UPD: This is **alpha** [cli command](https://cloud.google.com/sdk/gcloud/reference/alpha/iap/oauth-clients/reset-secret?hl=es_419%22&skip_cache=true) to reset client secret*:
```bash=1
gcloud alpha iap oauth-clients reset-secret NAME
```

* Reset API keys. ***Any project member can see your project's API key***

```bash=1
#List all API keys related to the specific projec
#A <Scope> filed can be a project, a folder, or an organization.
gcloud asset search-all-resources --scope=projects/PROJECT_ID --asset-types='apikeys.googleapis.com/Key'
```

>there's no CLI command to rotate API Keys, so use [credentials interface](https://console.cloud.google.com/apis/credentials/) to reset it manual
>
> *UPD: This is **alpha** [cli command](https://cloud.google.com/sdk/gcloud/reference/alpha/services/api-keys/create) to reset API keys*

### **3. Revoke access to VMs**

* Remove project-level SSH

```bash=1
#get ssh metadata project-level SSH keys used by the project
gcloud compute project-info describe --project=PROJECT_ID | Select-String ssh-keys -Context 5,5

#remove ssh metadata project-level SSH keys from the project
gcloud compute project-info remove-metadata --project=PROJECT_ID --keys=ssh-keys
```

* Remove instance-level SSH

```bash=1
#get ssh metadata instance-level SSH keys used by the VMs
gcloud compute instances describe --project=PROJECT_ID VM_INSTANCE | Select-String ssh-key -Context 5,5

#remove
gcloud compute instances describe --project=PROJECT_ID VM_INSTANCE | Select-String ssh-key -Context 5,5
```

> *Check for suspicious applications the person may have installed to provide backdoor access to the VM. If you are uncertain about the security of any code running on the VM, recreate it and redeploy the applications you need from source*

### **4. Revoke access to Cloud SQL databases**

* Find out SQL instances

```bash=1
#Find all Cloud SQL Instances or shrink scope with `--scope` attribute
gcloud asset search-all-resources --scope=<scope-area>/ID --asset-types='sqladmin.googleapis.com/BackupRun, sqladmin.googleapis.com/Instance'

#In Access Control (SQL instances intarface) check any misconfigurations
#and confirm that the list of IP addresses under Authorized networks and 
#list of apps under App Engine authorization match what you expect

#Click Users. In this tab, delete or change the password for any user 
#accounts the person had access to. 

#Be sure to update any applications that depend on those user accounts.
```

### **5. Find App Engine**

* App Engine apps have access by default to a service account that is an editor on the associated project. 
* App Engine request handlers can do things like create new VMs, and read or modify data in Cloud Storage. 
* Someone with the ability to deploy code to App Engine could use this service account to open a backdoor into your project. 
* If you're concerned about the code integrity of your deployed apps, you may want to redeploy them (including any modules) with a known-good checkout from your version control system

```bash=1
#List all App Engine
gcloud asset search-all-resources --scope=<scope-area>/ID --asset-types='appengine.googleapis.com/Service, appengine.googleapis.com/Version, memcache.googleapis.com/Instance'
```

### **6. Review and delete Cloud Storage bucket ACLs records**

* List all Cloud Storage

```bash=1
gcloud asset search-all-resources --scope=<scope-area>/ID --asset-types='storage.googleapis.com/Bucket' | Select-String bucket_name -Context 5,5

#go to project bucket and check ACLs settings
#If this settings contain fired user, remove this record
```

### **7. Revoke BigQuery dataset permissions**

```bash=1
#Find all BigQuery roles assigned to the fired user
gcloud asset search-all-iam-policies --scope=<scope-area>/ID --query="policy:fired_user@exapmle.com" --format="table(policy.bindings.user[], policy.bindings.role[])" | Select-String bigquery | Out-GridView

#Revome IAM policy according to the step one
```

### **8. Revoke Pub/Sub permissions**

```bash=1
#Find all Pub/Sub roles assigned to the fired user
gcloud asset search-all-iam-policies --scope=<scope-area>/ID --query="policy:fired_user@exapmle.com" --format="table(policy.bindings.user[], policy.bindings.role[])" | Select-String pubsub | Out-GridView

#Revome IAM policy according to the step one
```