# Azure Infrastructure Operations Project: Deploying a scalable IaaS web server in Azure

### Introduction
For this project, you will write a Packer template and a Terraform template to deploy a customizable, scalable web server in Azure.

### Getting Started
To use the code in this repository, you will follow a three-phase workflow:
1. **Prepare the environment:** Clone this repository locally and configure your Azure credentials as environment variables.
2. **Bake the Image:** Use Packer to create a custom Ubuntu machine image that contains our web application dependencies.
3. **Deploy the Infrastructure:** Use Terraform to read your customizable `vars.tf` variables and deploy a Virtual Machine Scale Set (VMSS), Networking, and Load Balancer architecture using the image created by Packer.

### Dependencies
1. Create an [Azure Account](https://portal.azure.com) 
2. Install the [Azure command line interface](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
3. Install [Packer](https://www.packer.io/downloads)
4. Install [Terraform](https://www.terraform.io/downloads.html)

### Instructions

**1. Authenticate with Azure**
Export your provided lab credentials as environment variables so Packer and Terraform can authenticate:
```bash
export ARM_CLIENT_ID="your-client-id"
export ARM_CLIENT_SECRET="your-client-secret"
export ARM_SUBSCRIPTION_ID="0c7436df-9359-4a04-b572-96ab6d89334c"
export ARM_TENANT_ID="f958e84a-92b8-439f-a62d-4f45996b6d07"

Note: If using a lab environment, export your provided credentials as environment variables (ARM_CLIENT_ID, ARM_CLIENT_SECRET, ARM_SUBSCRIPTION_ID, ARM_TENANT_ID).

**2. Enforce Tagging Policy**
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
  --resource-group "Azuredevops"

**3. Build the Server Image with Packer**
Navigate to the directory containing your Packer template to bake the application image.

# Validate the template
packer validate server.json

# Build the image
packer build server.json

Important: When the build completes, copy the Managed Image ID from the terminal output. You will need it for Terraform.

**4. Customize the Deployment (vars.tf)**
Before running Terraform, you can customize the deployment by modifying the vars.tf (or variables.tf) file.

Instance Count: Change the number of VMs in your scale set by updating the default value of the vm_count variable.

Location: Update the location variable to deploy the infrastructure to a different Azure region.

Resource Naming: Modify prefix variables to change how your Load Balancer, VNet, and VMSS are named in Azure.


**5. Provision Infrastructure with Terraform**
Deploy the infrastructure using your customized variables and the image created in Step 3.

# Initialize the Terraform working directory
terraform init

# Review the execution plan (replace with your actual Packer Image ID)
terraform plan -var="packer_image_id=<YOUR_MANAGED_IMAGE_ID>" -out=tfplan

# Apply the infrastructure
terraform apply tfplan

Note: Because of the policy applied in Step 2, ensure all Terraform resources in your configuration include a tags block, otherwise the deployment will be denied.

### Output

Upon successful deployment, Terraform will output the results to your terminal. The expected output includes:

Terminal Confirmation: A message stating Apply complete! Resources: X added, 0 changed, 0 destroyed.

Provisioned Resources in Azure:

   - A Virtual Network and Subnet.
   - A Network Security Group allowing external HTTP access.
   - A Public Load Balancer distributing traffic.
   - A Virtual Machine Scale Set spanning multiple VMs running your custom Packer image.

Application Verification: Terraform should output the Public IP address of your Load Balancer. If you copy that IP address and paste it into your web browser, you will see the "Hello, World!" web page served by the application you provisioned.
