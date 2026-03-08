# to-dos-cloud

## Getting Started
> Additional information about Yandex Cloud Terraform provider and Yandex Cloud resources can be found in documentation
 - [Yandex Cloud Terraform Provider](https://terraform-provider.yandexcloud.net)
 - [Yandex Cloud Documentation](https://yandex.cloud/en/docs)

### Create new folder in Yandex Cloud

1. Open [Yandex Cloud Console](https://console.yandex.cloud/cloud/)
1. Click the "Three dots" button placed next to the organization name at the top of the site
1. Click "Create folder" button
1. Specify the folder name `to-dos-prod`
1. Specify labels:
project: to-dos
environment: prod
1. Unselect "Create a default network" checkbox
1. Click "Create" button
1. When it is created switch to this folder in the left upper corner and proceed with the instruction in the new folder.

### Create S3 bucket for Terraform state

1. Open [Yandex Cloud Console](https://console.yandex.cloud)
1. Navigate to "Object Storage" service using the search (ALT + S)
1. Click "Create bucket" button
1. Specify the bucket name `to-dos-prod-terraform-state`, set the maximum size of the bucket to 1 GB and select "With authorization" option for all access settings.
1. Specify labels:
project: to-dos
environment: prod
1. Click "Create bucket" button

### Create Service Account

1. Open [Yandex Cloud Console](https://console.yandex.cloud)
1. Navigate to "Identity and Access Management" service using the search (ALT + S)
1. Click "Create service account" button
1. Specify the service account name `to-dos-prod-terraform-service-account` and add `storage.uploader`, `iam.editor`, `resource-manager.admin`, `compute.editor`, `vpc.publicAdmin`, `vpc.privateAdmin`, `vpc.securityGroups.admin` roles

>* `storage.uploader` to save terraform state in the created bucket
>* `iam.editor` to create additional service accounts
>* `resource-manager.admin` to give permissions to additional service accounts.
>* `compute.editor` to create and manage VM
>* `vpc.publicAdmin` to create public IP address
>* `vpc.privateAdmin` to create network, subnet and security group
>* `vpc.securityGroups.admin` to configure security group and it's incoming and outcoming connections

5. Specify labels:
project: to-dos
environment: prod
1. Click "Create" button

### Create access keys

1. Open [Yandex Cloud Console](https://console.yandex.cloud)
1. Navigate to Identity and Access Management service using the search (ALT + S)
1. Select the created service account
1. Click the "Create new key" button to allow to create cloud resources.
1. In the menu that appears click "Create authorized key" button
1. Select "RSA_4096" Encryption algorithm
1. Click "Create" button
1. Click "Download file with keys" the created key and copy it to this repo folder.
1. Click "Close" button
1. Click the "Create new key" button to allow to write to `to-dos-prod-terraform-state` bucket.
1. In the menu that appears click "Create static access key" button
1. Click "Create" button
1. Create a new file in this repo folder, name it "config.s3.tfbackend" and paste the created access key like in the following example:
```bash
access_key="PASTE_YOUR_ACCESS_KEY_HERE"
secret_key="PASTE_YOUR_SECRET_KEY_HERE"
```

### VSCode Dev Container

Open this repo's folder in VSCode, it might immediately propose you to re-open it in a Dev Container or you can click on Remote Explorer, find plus button and choose the Open Current Folder in Container option and wait when it is ready.

When your Dev Container is ready, the VS Code window will be re-opened. Open a new terminal in this Dev Container which will be executing the commands under this prepared Linux container where we have already pre-installed and pre-configured:
- Terraform to automatically create resources in the Yandex Cloud

### Create SSH key

To create SSH that will be used to log in into VM execute the following command:
> Note: SSH key should be named as project name + environment + ssh. Example: to-dos-prod-ssh

```bash
ssh-keygen -t ed25519 -f ./to-dos-prod-ssh
```

Specify strong passphrase. Don't leave it empty!

## Terraform

### Terraform initialization

To initialize terraform provider execute the following command:
```bash
terraform init -backend-config=config.s3.tfbackend -backend-config="bucket=to-dos-prod-terraform-state"
```
> By default terraform was configured for amd64 platform. To add support of arm64 CPU's to `.terraform.local.hcl` file we used the following command:
```bash
terraform providers lock -platform=linux_arm64
```

### Terraform validation

To validate that your terraform configuration was written right execute the following command:
> terraform provider should be initialized
```bash
terraform validate
```

### Configuring terraform variables

1. Copy terraform.tfvars.example to a new file terraform.tfvars (it is in .gitignore)
2. Change <TO_BE_MODIFIED!!!> in terraform.tfvars to your values

> How to find Cloud ID and Folder ID
> 1. Open [Yandex Cloud Center](https://center.yandex.cloud)
> 1. Select your folder from the list on the left.
> 1. Cloud ID located near to the organization name at the top of the site.
> 1. Folder Id located near to the folder name at the top of the site.

3. Rename terraform.tfvars.example to terraform.tfvars

To apply terraform configuration execute the following command:
```bash
terraform apply -auto-approve
```
