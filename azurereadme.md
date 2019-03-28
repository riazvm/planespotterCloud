# planespotterCloud
Cloned from https://github.com/yfauser/planespotter

# Provision Instances and depoloy the planespotter App on AWS using VMWare Cloud Assembly

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


- az network nsg rule create -g CASResourceGroup --nsg-name CASPublicNSG -n PublicNSGRule --priority 100 \
                            --destination-address-prefixes '*' --destination-port-ranges 80 --access Allow \
                            --protocol Any --description "Allow"

- az network nsg rule create -g CASResourceGroup --nsg-name CASPublicNSG -n PublicNSGRule --priority 120 \
                            --destination-address-prefixes '*' --destination-port-ranges 443 --access Allow \
                            --protocol Any --description "Allow"

- az network nsg rule create -g CASResourceGroup --nsg-name CASPublicNSG -n PublicNSGRule --priority 130 \
                            --destination-address-prefixes '*' --destination-port-ranges 22 --access Allow \
                            --protocol Any --description "Allow"

###### NSG for Private Subnet
- az network nsg create -g CASResourceGroup -n CASPrivateNSG -l centralus

##### Associate the NSG to the subnets
- az network vnet subnet update -g CASResourceGroup -n PublicSubnet --vnet-name CASVirtualNetwork --network-security-group CASPublicNSG
- az network vnet subnet update -g CASResourceGroup -n PrivateSubnet --vnet-name CASVirtualNetwork --network-security-group CASPrivateNSG



### Create a Key Pair
- Click on services and Search for EC2 and select EC2
- On the left menu under Network and Security select KeyPairs
- Click on KeyPairs and enter name as planeskeypair
- A planeskeypair.pem file is generated. Store this file in a safe location do not loose this file.


###### NOTE: You can only access the compute instances in the private subnet from the public subnet

### Provision a Bastion Host
- Click on services and Search for EC2.
- Launch an instance, select a windows machine which is free tier eligible (Eg. Microsoft Windows Server 2019 Base - ami-0410d3d3bd6d555f4)
- Select Instance Type as t2 micro (Free tier eligible) and click on configure instances
- Select Network as vpc -xxx | Dev
- Select subnet as Dev Public 
- Click on Add Storage, and Click on Tags (Key Name , Value Bastion Host)
- Configure Security Group , create new , give the security group a name, by default RDP should be enabled
- Click Review and launch, Select existing keypair (planeskeypair)
- Acknowledge and launch the instance


### Delete AWS Stack

- Terminate all instances running in the VPC
- Search for CloudFormation under services
- Select the Stack that was created (Eg. mycasoneazstack) 
- Click on Actions Delete Stack

 


## VMWare Cloud Services 
Login to your CAS Account and select VMware Cloud Assembly

### Configure Infrastructure

#### Cloud Account
- Click on Infrastructure and add select Cloud Accounts under Connections
- Click on Add Cloud Account
- Select Amazon Webservices
- Enter AccessKey and AccessKey (this is available in the credentials file that was downloaded while provisioning a user)
- Click on Validate
- Enter a Name for the account, description and tag and click on ADD

### Projects
- Click on Projects under Configure
- New Project (Eg. Planes)

### Cloud Zones
- New Cloud Zone
- Select account that was created in the previous step
- Enter a Name (PlanesCloudZone0, Description
- For Capabilitytags enter values Eg(planespublicsubnet, planesprivatesubnet). We will be using these tags to differentiate between the public and private subnets in our vpc. Click on create
- For to Projects , open Panes, go to provisioning and add the PlanesCloudZone to the project and save
		
### Flavor Zones
- By default there is a small , large and medium. You can add more flavor mappings , by adding a new flavor mapping and selecting your account and selecting the type of instance you require.

### Image Mapping
- Add new Image mapping , enter a image name, select account , for the image enter the ami id , select the ami and save (Eg. AmazonLinuxAMI (ami-0080e4c5bc078760e), AWS Ubuntu (ami-0f9cf087c1f27d9b1))

### Network Profile
- Add New Network Proifile
- Select Account, enter name (Eg. PlanesNetworkProfile)
- In networks select the Dev Public Subnet(AZ) and the Dev Public Subnet(AZ), these were created in your VPC when it was provisioned
- Select the Dev Public Subnet and click on tags and enter planespublicsubnet and save
- Select the Dev Private Subnet and click on tags and enter planesprivatesubnet and save
NOTE: These tags are used to provision the compute intances to their respective Subnets
- Click on create

### BluePrints
- Go to blueprints and create a new blue print
- Enter Name(Eg PlanesVPCBluePrint) and select the project as planes
- Copy the contents of the planespotterCloud/planespotter-master/vmwarecas/blueprints/PlanesVPCBluePrint.yaml to the new blueprint
- Go through the contents on the yaml (note the tags used for the private and public subnets)
- Deploy the blueprint


## Verification and Configuration Changes

### EC2 Instances
- Go to the EC2 dashboard . Log into AWS , serach for EC2 under services
- You should see four new EC2 instances running in your dashboard
- Under the EC2 Dashboard click on Security Groups. CAS would have created the photon-model-sg security group.
- Click on the API, DB and Cache instances and make sure their subnet is the private subnet of your vpc
- make sure that the frontend instance is on the public subnet of your vpc
- Get private IP of all instances
- select each instance and copy the private IP to your clipboard

### Connect to the instances:

### Windows users:
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html

### Front end Instance
- In a terminal window navigate to the folder where the planeskeypair.pem was downloaded
- Go to your EC2 dashboard and select the frontend instance and click on connect 
- Follow the instructions in the popup (chmod 400 planeskeypair.pem)
- copy the string under Example (ssh -i "planeskeypair.pem" ubuntu@<ip>)
- Paste the string in your terminal and click on enter
- cd /planespotterCloud/planespotter-master/frontend/app/
- edit frontend.ini
- Point the PLANESPOTTER_API_ENDPOINT endpoint to your API instance private ip (10.192.20.x)
- Restart the frontend service (sudo systemctl restart frontend)

### API Instance
-RDP into the bastion Host 
- Go to the EC2 dashboard . Log into AWS , serach for EC2 under services
- Select the BastionHost and copy its public IP to clip board
- Click on Connect and follow the instructions to get password.
- Select the planeskeypair.pem file as key file
- Download the RDP file
- RDP to the Bastion host using the username and password 
- Transfer the Pem file to the bastion host
- Follow the instructions on https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html to connect to the api instance (NOTE: Connect to the API instance using the private IP 10.192.20.x)
- use ubuntu@10.192.20.x for hostname or ip
- On the putty terminal cd /planespotterCloud/planespotter-master/app-server/app/config/
- edit the config.cfg file and change the DATABASE_URL ip and REDIS_HOST ip to the private ip of the DB and Cache Instace's
- Restart the app-server service (sudo systemctl restart app-server)


## Check Planespotter Application
- Open a Browser and enter the IPV4 public ip of the front end server on your browser. 
- Navigate to APP Health and check if all components of the app show green
