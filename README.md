# Deploying-a-scalable-IaaS-Web-Server-in-Azure

The following README chronologically guides you through the steps to create and to deploy a scalable IaaS web server in  Azure. Clone the repository and run the commands following the instructions of this README.

*Result: scalable IaaS web server in Azure*


## Preconditions
Before you get started ensure you performed the following preconditions to be able to deploy the IaaS web server in Azure:
* Create an [Azure account](https://portal.azure.com)
* Install the [Azure command line interface](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
* Install [Packer](https://www.packer.io/downloads)
* Install [Terraform](https://www.terraform.io/downloads.html)

Verify the installation with the following commands:
* `az —version`, expected to return 2.15.1 or higher
* `packer —version`, expected to return 1.6.5 or higher.
* `terraform —version`, expected to return v0.14.0 or higher.

## Variables
The vars.tf file contains the following variables which you can change by adding or changing the default in the file:
* *tenant_id*: Tenant_id. You will need to add your tenand id, before deploying the Infrastructure as code with terraform. Run `az account show`, in the command line and add your tenant id as default to the vars.tf file.
* *project*: This will be the name of the resource group that keeps the majority of resources (subnet, virtual machines,...). It is also a tag-value; each resource that can be tagged will have this project tag (must be enforced by policy to be sure!). This way we can track resources associated with this project.
* *location*: The Azure region in which all resources will be created.
* *prefix*: The prefix, which will be applied to all resources.
* *virtual_machine_number*: Number of virtual machines which will be created.
* *image_id*: Image Id, created with Packer (see section 2, bullet point 2). Note: You are either required to add a default value to the vars.tf file, before deploying the Infrastructure with terraform or you will be asked to enter the Image Id, when running terraform.apply. After you created the Image with Packer, you will get the Image ID by running `az image list` and by looking for the id : /subscriptions/...providers/Microsoft.Compute/...".
* *admin_username*: This is the name of the administrator of the virtual machines.
* *admin_password*: This is the administrator password required for logon the virtual machines.

## Step-by-Step Instruction
The following instructions guide you step-by-step through the creation & deployment of the IaaS webserver in Azure. Ensure the preconditions listed in the section above before getting started.

### 1. Create service principles for Terraform & Packer
Create service principles for Terraform & Packer.

*For Packer:*
* Run `az account show —query „{ subscription_id: id}"`.
* Create the following environment variable using the output of the last command: ARM_SUBSCRIPTION_ID
* Run `az ad sp create-for-rbac --query "{ client_id: appId, client_secret: password, tenant_id: tenant }"`.
* Use the output of the last comment to create the following environment variables: ARM_CLIENT_ID and ARM_CLIENT_SECRET.

*For Terraform:*
* Login to your Azure account with `az login`.
* Run `az account show`, to get the tenant id. Copy the tenant id.
* Paste it as default =„" for tenat_id in the vars.tf file (line 1).
* Run `terraform init`, to initialize Terraform. Successful init returns „Terraform has been successfully initialized!“

### 2. Create your infrastructure as code
This section explains how to provision the infrastructure for the web server in Azure.

* Create the resource group for the custom image with the following command:
`az group create --name images-4-webapp --location "West Europe" --tags "project=udacity"`
* Create & deploy the custom image with Packer by running `packer build server.json`. This will take a couple of minutes. List the image you just created with `az image list.`
* Copy the image id from the output above and add it as default for the  variable "image_id" in the vars.tf file (at line 26).
* Run `terraform plan -out solution.plan` to plan the infrastructure for being deployed in Azure.
* Run `terraform apply` to create the infrastructure in Azure. Note: the creation will take a few minutes.

### 3. Check the deployed infrastructure
 Verify the results in the portal: navigate to `resource groups`, and click on the resource group `udacity`, to check the resources which you just created.
 To view the image, click on the image `images-4-webapp`.
 You can also run `terraform.show`to view the deployed infrastructure in json format.

### 4. Delete the infrastructure
To delete the infrastructure you need to both delete the stack created with terraform and the image created with packer.
* Run `terraform.destroy`to delete the infrastructure with terraform.
* Run `az image delete -g images-4-webapp -n image`to delete the image in Azure.
* Finally run `az group delete --resource-group images-4-webapp -y`to delete the resource group for the image.

*You successfully created, deployed, checked and deleted the IaaS-Web-Server-in-Azure!*
