
# Provision Instances and depoloy the planespotter App on GCP using VMWare Cloud Assembly

- Create VPC with a public and private Subnet on a single AZ.  Deploys an Internet Gateway, with a default route on the public subnet. Deploy a NAT Gateways and default routes for it in the private subnets.

- Create an service account within GCP to connect from CAS

- Create and provision a front end server in the public subnet , an api server, a cache server and a mysql db server in the private subnet.

- CAS creates a bastion host on the public subnet to connect and validate the instances on the private subnet

- Change IP addresses on the configuration file on the API  Server to point to the dbserver and cache server .

- Change the IP address on the ini file on the frontend server to point to the API server.

##  Application Architecture
![](https://github.com/riazvm/planespotterCloud/blob/master/planespotter-master/docs/pics/GCPVPNPlanespotter.png)

## Google Cloud CLI
 - https://cloud.google.com/sdk/docs/

## Setup a Project
- Login to GCP 
- Select dropdown next to "GOOGLE CLOUD PLATFORM"
- Create a new project ands select the org to which the project is to be assigned to.
- Alternatively follow instructions here https://cloud.google.com/resource-manager/docs/creating-managing-projects

##  Create VPN (Virtual Private Network)
- Follow instructions on  https://cloud.google.com/vpc/docs/using-vpc to create a VPC
- Create two subnets azplanespubsubnet and azplanespvtsubnet

Note: Firewall rules - Make sure you enable the subnets to tralk with each other by creating firewall rules https://cloud.google.com/vpc/docs/firewalls


### Create a Service account
- Login to the google cloud Portal
- Search for IAM on the search bar
- Select IAM & admin and click on Service accounts
- Create a service account and give project Editor role and int he final screen create a key of type json.
- A json private key will be generated and will automatically download to your computer.
- Save this json key we will be using this when we create a cloud account in CAS


## VMWare Cloud Services 
Login to your CAS Account and select VMware Cloud Assembly

### Configure Infrastructure

#### Cloud Account
- Click on Infrastructure and add select Cloud Accounts under Connections
- Click on Add Cloud Account
- Select Google Cloud Platfoem
- Click on Import Json key and select the key generated when creating a servive account.	
- Click on Validate
- Enter a Name for the account, description and tag and click on ADD
- Select the region Eg. us-cental1
- Click on Add

### Projects
- Click on Projects under Configure
- New Project (Eg. Planes)
- Alternatively Add to an Existing Project by clicking on the project , Provisioning and add cloud zone that was automatically created by CAS for GCP

### Cloud Zones
- Select the cloud zone that was provisioned by CAS for Google. Add azplanespubsubnet and azplanespvtsubnet to the capabilites tag of the cloud zone.

### Flavor
- By default there is a small , large and medium. You can add more flavor mappings , by adding a new flavor mapping and selecting your account and selecting the type of instance you require.
- Click on the + , select the account and enter value eg Standard_B1s and click on save

### Image Mapping
- Add new Image mapping , enter a image name, select account and Select the Image Eg Canonical:UbuntuServer:18.04-LTS:latest and click on create. You can also add to an existing image.

### Network Profile
- Add New Network Proifile
- Select Account, enter name (Eg. GCPPlanesNetworkProfile)
- In networks click on add and select the PublicSubnet and the PrivateSubnet, these were created in your VPC when it was provisioned , and click on Add
- Select the PublicSubnet and click on tags and enter azplanespubsubnet and save
- Select the Dev PrivateSubnet and click on tags and enter azplanespvtsubnet and save
NOTE: These tags are used to provision the compute intances to their respective Subnets
- Click on create

### BluePrints
- Go to blueprints and create a new blue print
- Enter Name(Eg PlanesVPCBluePrint) and select the project as planes
- Copy the contents of the planespotterCloud/planespotter-master/vmwarecas/blueprints/cloudagnostic/planespottercloudagnostic.yaml to the new blueprint
- Go through the contents on the yaml (note the tags used for the private and public subnets)
- Deploy the blueprint


## Verification and Configuration Changes

### GCP VM Instances
- Login to Google Cloud Platform
- Click on Compute Engine - VM Instances
- You should see 5 virtual machines provisioned , Bastion, frontend, api, cache and DBTier

## Check Planespotter Application
- Open a Browser and enter the IPV4 public ip of the front end server on your browser. 
- Navigate to APP Health and check if all components of the app show green
