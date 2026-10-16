# Azure Infrastructure Operations Project: Deploying a scalable IaaS web server in Azure

### Introduction
For this project, you will write a Packer template and a Terraform template to deploy a customizable, scalable web server in Azure.

### Getting Started
1. Clone this repository
2. Create your infrastructure as code
3. Update this README to reflect how someone would use your code.

### Dependencies
1. Create an [Azure Account](https://portal.azure.com) 
2. Install the [Azure command line interface](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
3. Install [Packer](https://www.packer.io/downloads)
4. Install [Terraform](https://www.terraform.io/downloads.html)

### Instructions
**1. Authenticate with Azure**
Before running any tools, you must authenticate your terminal with your Azure account. You can do this by running:
```bash
az login

Note: If using a lab environment, export your provided credentials as environment variables (ARM_CLIENT_ID, ARM_CLIENT_SECRET, ARM_SUBSCRIPTION_ID, ARM_TENANT_ID).

2. Enforce Tagging Policy
To comply with security and organizational standards, deploy the Azure Policy that denies the creation of resources without tags.

# Create the policy definition
az policy definition create \
  --name "deny-resources-without-tags" \
  --rules deny-no-tags-rule.json \
  --mode Indexed

# Assign the policy to your resource group
az policy assignment create \
  --name "require-tags-assignment" \
  --policy "deny-resources-without-tags" \
  --resource-group <YOUR_RESOURCE_GROUP_NAME>

3. Build the Server Image with Packer
Bake the application and its dependencies into a custom Managed Image.

# Validate the template
packer validate server.json

# Build the image
packer build server.json

Important: When the build completes, copy the Managed Image ID from the terminal output. You will need it for Terraform.

4. Provision Infrastructure with Terraform
Deploy the Virtual Machine Scale Set (VMSS), Load Balancer, and Networking resources.

# Initialize the Terraform working directory
terraform init

# Review the execution plan (replace with your actual Packer Image ID)
terraform plan -var="packer_image_id=<YOUR_MANAGED_IMAGE_ID>" -out=tfplan

# Apply the infrastructure
terraform apply tfplan

Note: Because of the policy applied in Step 2, ensure all Terraform resources in your configuration include a tags block, otherwise the deployment will be denied.

### Output

Once the Terraform deployment is complete, the following infrastructure will be provisioned in your Azure Resource Group:

Custom Managed Image: A golden Ubuntu image containing the pre-configured web application.

Networking: A Virtual Network (VNet) and Subnet.

Security: A Network Security Group (NSG) restricting incoming traffic to necessary ports (e.g., HTTP).

Load Balancing: A Public Load Balancer with a Public IP address to distribute incoming traffic.

Compute: A scalable Virtual Machine Scale Set (VMSS) running the custom Packer image behind the load balancer.

You will be able to retrieve the Load Balancer's Public IP from the Azure Portal (or via Terraform outputs) and navigate to it in your web browser to see the running web application.
