# Overview:  JuliaRT
Pentest Cyber Range for a small Active Directory Domain.  Automated templates for building your own Pentest/Red Team/Cyber Range in the Azure cloud!  JuliaRT is a small Active Directory enterprise deployment automated with Terraform / Ansible Playbook templates to be deployed in Azure.
# Quick Fun Facts:
* Deploys one (1) Windows 2019 Domain Controller and three (3) Windows 10 Pro Endpoints
* Automatically joins the three Windows 10 computers to the AD Domain
* Uses Terraform templates to automatically deploy in Azure with VMs
* Terraform templates write Ansible Playbook configuration, which can be customized
* Post-deploy Powershell script that adds registry entries on each Windows 10 Pro endpoint to automatically log in each username into the Domain as respective user
* Automatically uploads Badblood (but does not install) if you prefer to generate thousands of simulated users https://github.com/davidprowe/BadBlood
* Post-deployment Powershell script provisions three domain users on the 2019 Domain Controller and can be customized for many more
* Domain Users:  olivia (Domain Admin); lars (Domain User); liem (Domain User)
* All Domain User passwords:  Password123
* Domain:  RTC.LOCAL
* Domain Administrator Creds:  RTCAdmin:Password123
* Deploys four IP subnets
* Deploys intentionally insecure Azure Network Security Groups (NSGs) that allow RDP, WinRM (5985, 5986) from the Public Internet.  Secure this as per your requirements.  WinRM is used to automatically provision the hosts.

# JuliaRT Deployment Instructions
**Note:**  Tested on Ubuntu Linux 20.04 

Requirements:
* Azure subscription
* Terraform:  Tested on v0.12.26
* Ansible:  Tested on 2.9.6

## Installation Steps

**Note:**  Tested on Ubuntu 20.04

**Step 1:** Install Terraform and Ansible on your Linux system

Download and install Terraform for your platform --> https://www.terraform.io/downloads.html

Install Ansible
```
$ sudo apt-get install ansible
```

**Step 2:** Set up an Azure Service Principal on your Azure subscription that allows Terraform to automate tasks under your Azure subscription

Follow the exact instructions in this Microsoft link:
https://docs.microsoft.com/en-us/azure/developer/terraform/getting-started-cloud-shell

These were the two basic commands that were run based on this link above:
```
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<subscription_id>
```
and this command below.  From my testing I needed to use a role of "Owner" instead of "Contributor".  Default Microsoft documentation shows role of "Contributor" which resulted in errors.  
```
az login --service-principal -u <service_principal_name> -p "<service_principal_password>" --tenant "<service_principal_tenant>"
```
Take note of the following which we will use next to configure our Terraform Azure provider:
```
subscription_id = ""
client_id = ""
client_secret = ""
tenant_id = ""
```

**Step 3:** Clone this repo
```
$ git clone https://github.com/iknowjason/juliart.git
```

**Step 4:** Using your favorite text editor, edit the terraform.tfvars file for the Azure resource provider matching your Azure Service Principal credentials

```
cd juliart/deploy
vi terraform.tfvars
```

Edit these parameters in the terraform.tfvars file:
```
subscription_id = ""
client_id = ""
client_secret = ""
tenant_id = ""
```

Your terraform.tfvars file should look similar to this but with your own Azure Service Principal credentials:
```
subscription_id = "aa9d8c9f-34c2-6262-89ff-3c67527c1b22"
client_id = "7e9c2cce-8bd4-887d-b2b0-90cd1e6e4781"
client_secret = ":+O$+adfafdaF-?%:.?d/EYQLK6po9`|E<["
tenant_id = "8b6817d9-f209-2071-8f4f-cc03332847cb"
```

**Step 5:** Run the commands to initialize terraform and apply the resource plan

```
$ cd juliart/deploy
$ terraform init
$ terraform apply -var-file=terraform.tfvars -auto-approve
```

This should start the Terraform automated deployment plan

**Step 6:** Optional:  Unzip and run Badblood from C:\terraform directory (https://github.com/davidprowe/BadBlood)

# Known Issues or Bugs
Sometimes the Windows 10 Endpoints don't automatically log into the domain via registry entry.  I've traced this issue to a timing issue with the Domain Controller creation.  The powershell script creating the three users does not run correctly.  To resolve the issue, simply run the Ansible Playbooks in each module directory.  The following should resolve the issue:
```
$ cd ../modules/dc1-vm/
$ ansible-playbook -i hosts.cfg playbook.yml

$ cd ../win10-vm-1/
$ ansible-playbook -i hosts.cfg playbook.yml

$ cd ../win10-vm-2/
$ ansible-playbook -i hosts.cfg playbook.yml

$ cd ../win10-vm-3/
$ ansible-playbook -i hosts.cfg playbook.yml
```

![](images/Julia_Image1.jpeg)

# Credits

@ghostinthewires for his Terraform templates (https://github.com/ghostinthewires)

@mosesrenegade for his Ansible Playbook integration with Terraform + Powershell script (https://github.com/mosesrenegade)

@davidprowe for his Badblood (https://github.com/davidprowe/BadBlood)
