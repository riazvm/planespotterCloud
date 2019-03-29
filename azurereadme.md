
# Provision Instances and depoloy the planespotter App on Azure using VMWare Cloud Assembly

- Create VPC with a public and private Subnet on a single AZ.  Deploys an Internet Gateway, with a default route on the public subnet. Deploy a NAT Gateways and default routes for it in the private subnets.

- Create an applciation within Azure to connect from CAS

- Create and provision a front end server in the public subnet , an api server, a cache server and a mysql db server in the private subnet.

- CAS creates a bastion host on the public subnet to connect and validate the instances on the private subnet

- Change IP addresses on the configuration file on the API  Server to point to the dbserver and cache server .

- Change the IP address on the ini file on the frontend server to point to the API server.

##  Application Architecture
![](https://github.com/riazvm/planespotterCloud/blob/master/planespotter-master/docs/pics/AzureVPNPlanespotter.png)

##  Create VPN (Virtual Private Network)

### Install the Azure CLI
- https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest

### Configure Azure CLI
- open a terminal window and enter az login (Browser should open up with prompt)
- Login with your azure account on the browser

### Provision the VPN (Virtual Private Network)
#### Through Portal
- https://docs.microsoft.com/en-us/azure/virtual-network/
- https://www.youtube.com/watch?v=ADdGZEfmNzQ

#### Through CLI

##### Create a resource group
- az group create -l centralus -n CASResourceGroup

##### Create a Virtual Network and a Public Subnet
- Create a virtual network with a specific address prefix and a Public subnet.

- az network vnet create -g CASResourceGroup -n CASVirtualNetwork --address-prefix 10.0.0.0/23 --subnet-name PublicSubnet --subnet-prefix 10.0.0.0/24

##### Create a Private Subnet

- az network vnet subnet create --address-prefixes 10.0.1.0/25 --name PrivateSubnet --resource-group CASResourceGroup --vnet-name CASVirtualNetwork

##### Add a Gateway Subnet
###### NOTE: NOT NECESSARY  - Used to connect a Virtual Private Network in AZURE to your corporate network 

- az network vnet subnet create --vnet-name CASVirtualNetwork -n GatewaySubnet -g CASResourceGroup --address-prefix 10.0.1.128/28 

##### Create a Network Security group
###### NOTE: NSG are firewalls that have both inbound and outbound rules that are processed from lowest to highest. 

###### NSG For the Public Subnet
- az network nsg create -g CASResourceGroup -n CASPublicNSG  -l centralus

###### Add rule to allow all HTTP , HTTPS  and SSH traffic into the public subnet


- az network nsg rule create -g CASResourceGroup --nsg-name CASPublicNSG -n PublicNSGRuleHTTP --priority 100 --destination-address-prefixes '*' --destination-port-ranges 80 --access Allow --protocol TCP --description "Allow"

- az network nsg rule create -g CASResourceGroup --nsg-name CASPublicNSG -n PublicNSGRuleHTTPS --priority 120 --destination-address-prefixes '*' --destination-port-ranges 443 --access Allow --protocol TCP  --description "Allow"

- az network nsg rule create -g CASResourceGroup --nsg-name CASPublicNSG -n PublicNSGRuleSSH --priority 130  --destination-address-prefixes '*' --destination-port-ranges 22 --access Allow  --protocol TCP --description "Allow"

###### NSG for Private Subnet
- az network nsg create -g CASResourceGroup -n CASPrivateNSG -l centralus

##### Associate the NSG to the subnets
- az network vnet subnet update -g CASResourceGroup -n PublicSubnet --vnet-name CASVirtualNetwork --network-security-group CASPublicNSG
- az network vnet subnet update -g CASResourceGroup -n PrivateSubnet --vnet-name CASVirtualNetwork --network-security-group CASPrivateNSG



### Create a Application in Azure
- Login to the azureportal (portal.azure.com)
- Navigate to Azure Active Directory  App Registrations (Preview)
	+ New Registrations 
- Enter A Name planeSpotterCAS

##### Supported AccountTypes
- Select Accounts in any organizational directory and personal Microsoft accounts (e.g. Skype, Xbox, Outlook.com)
- Select Dropdown as Public Client
- Redirect URL : urn:ietf:wg:oauth:2.0:oob
- Click on register

##### Certificate and Secrets
- Select Certificate and Secrets
- Click on +New client secret
- Description  “plane spotter app key”
- Expires as Never
- Click on Add
- Copy the value to a clip board (This is the client Secret)

##### Important ID's
- Client Secret - Copy the value from above clip board
- Go to overview and select the Application(Client) ID and Directory(tenant) ID to a clipboard
- Click on All services in the left hand menu
- Under General click on Subscriptions 
- Select Subscription that you are using

##### Assign Role to App 
- Click on AccessControl(IAM)
- Click on Add a Role Assignment
- Select Role as Contributor
- Under Select search for the app you had created Eg. planeSpotterCAS
- Select the app and click on Save

### Delete Azure Stack

- Login to Azure portal (portal.azure.com)
- Click on Resource Groups
- Select the resource group (Eg CASResourceGroup)
- Delete Resource Group

## VMWare Cloud Services 
Login to your CAS Account and select VMware Cloud Assembly

### Configure Infrastructure

#### Cloud Account
- Click on Infrastructure and add select Cloud Accounts under Connections
- Click on Add Cloud Account
- Select Azure
- Enter Subscription ID,Tenant ID,Client application ID, Client application secret key from the step abpve
- Click on Validate
- Enter a Name for the account, description and tag and click on ADD
- Select the region Eg. Central US
- Click on Add

### Projects
- Click on Projects under Configure
- New Project (Eg. Planes)
- Alternatively Add to an Existing Project by clicking on the project , Provisioning and add cloud zone that was automatically created by CAS for Azure

### Cloud Zones
- Select the cloud zone that was provisioned by CAS for Azure. Add azplanespubsubnet and azplanespvtsubnet to the capabilites tag of the cloud zone.

### Flavor
- By default there is a small , large and medium. You can add more flavor mappings , by adding a new flavor mapping and selecting your account and selecting the type of instance you require.
- Click on the + , select the account and enter value eg Standard_B1s and click on save

### Image Mapping
- Add new Image mapping , enter a image name, select account and Select the Image Eg Canonical:UbuntuServer:18.04-LTS:latest and click on create. You can also add to an existing image.

### Network Profile
- Add New Network Proifile
- Select Account, enter name (Eg. AzurePlanesNetworkProfile)
- In networks click on add and select the PublicSubnet and the PrivateSubnet, these were created in your VPC when it was provisioned , and click on Add
- Select the PublicSubnet and click on tags and enter azplanespubsubnet and save
- Select the Dev PrivateSubnet and click on tags and enter azplanespvtsubnet and save
NOTE: These tags are used to provision the compute intances to their respective Subnets
- Click on create

### BluePrints
- Go to blueprints and create a new blue print
- Enter Name(Eg PlanesVPCBluePrint) and select the project as planes
- Copy the contents of the planespotterCloud/planespotter-master/vmwarecas/blueprints/azure/PlanesVPCBluePrintAzure.yaml to the new blueprint
- Go through the contents on the yaml (note the tags used for the private and public subnets)
- Deploy the blueprint


## Verification and Configuration Changes

### Azure VM Instances
- Login to Azure Portal
- Click on All resources
- Select Filters
- Select the resourcegroup Eg. CASResourceGroup
- Select the resources Type as Virtual Machines
- You should see 5 virtual machines provisioned , Bastion, frontend, api, cache and DBTier

### Connect to the instances:

### Windows users:
Use Putty to ssh into host machines

### Front end Instance
- Select the frontend vm and copy the public ip
- SSH into the frontend vm eg ssh ubuntu@publicip (password in the blueprint is Vmware123456)
- cd /planespotterCloud/planespotter-master/frontend/app/
- edit frontend.ini
- Point the PLANESPOTTER_API_ENDPOINT endpoint to your API instance private ip (10.192.20.x)
- Restart the frontend service (sudo systemctl restart frontend)

### API Instance
- SSH into the bastion Host 
- Select the bastion vm and copy the public ip in the azure portal
- SSH into the bastion vm eg ssh ubuntu@<bastionpubklicip> (password in the blueprint is Vmware123456)
- Select the api vm and copy the private ip from the azure portal
- From the bastion vm SSH into the private vm eg ssh ubuntu@<apiprivateip> (password in the blueprint is Vmware123456)
- On the putty terminal cd /planespotterCloud/planespotter-master/app-server/app/config/
- edit the config.cfg file and change the DATABASE_URL ip and REDIS_HOST ip to the private ip of the DB and Cache Instace's
- Restart the app-server service (sudo systemctl restart app-server)


## Check Planespotter Application
- Open a Browser and enter the IPV4 public ip of the front end server on your browser. 
- Navigate to APP Health and check if all components of the app show green
