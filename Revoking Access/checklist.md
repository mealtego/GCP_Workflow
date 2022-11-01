
# Check List. Revoking Access to Google Cloud Platform

## According to officail [Google guidlance](https://cloud.google.com/docs/security/data-loss-prevention/revoking-user-access#revoking_access)

### **1. Remove the account from project membership**

```bash=1
# Find all **IAM policies** granted to the specific account
gcloud asset search-all-iam-policies --scope=organizations/820076494741 --query="policy:fired_user@exapmle.com" --sort-by=resource --format="table(assetType, resource, policy.bindings.role[])"

# Remove **IAM policy** granted to the specific principal
gcloud resource-manager <RESOURCE_TYPE> remove-iam-policy-binding 355580441671  --member=user:fired_user@example.com --role=roles/<fired_user_role>
```

### **2. Rotate project credentials**

* Rotate Service account keys. ==If the person whose access is being revoked was not an owner, you may skip this step==

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

* Reset OAuth 2.0 client ID secrets. ==If the person whose access is being revoked was not an owner or editor, you may skip this step==

>there's no CLI command to reset OAuth client secret, so use [credentials interface](https://console.cloud.google.com/apis/credentials/) to reset it manual
>
> *UPD: This is **alpha** [cli command](https://cloud.google.com/sdk/gcloud/reference/alpha/iap/oauth-clients/reset-secret?hl=es_419%22&skip_cache=true) to reset client secret*:
```bash=1
gcloud alpha iap oauth-clients reset-secret NAME
```

* Reset API keys. ==Any project member can see your project's API key==

```bash=1
#List all API keys related to the specific projec
#A <Scope> filed can be a project, a folder, or an organization.
gcloud asset search-all-resources --scope=projects/PROJECT_ID --asset-types='apikeys.googleapis.com/Key'
```

>there's no CLI command to rotate API Keys, so use [credentials interface](https://console.cloud.google.com/apis/credentials/) to reset it manual
>
> *UPD: This is **alpha** [cli command](https://cloud.google.com/sdk/gcloud/reference/alpha/services/api-keys/create) to reset API keys*

### **3. Revoke access to VMs**