# CAS Planespotter Deployment Architecture

## Planespotter App

Planespotter is composed of a MySQL DB that holds Aircraft registration data from the FAA. You can search through the data in the DB through an API App Server written in Python using Flask. The API App Server is also retrieving data from a Redis in memory cache that contains data from aircrafts that are currently airborne. There's a service written in Python that retrieves the Data about airborne aircrafts and pushes that data into Redis. Finally there is a Frontend written with Python Flask and using Bootstrap.

![](https://github.com/riazvm/planespotterCloud/blob/master/planespotter-master/docs/pics/planespotter.png)

## CAS Planespotter Deployment Topology

![](https://github.com/riazvm/planespotterCloud/blob/master/planespotter-master/docs/pics/topology.png)

- The application topology shows the different systems and networks that consitute the planespotter application

- The planespotter application is deployed as a multitier application with a clear segregation of each app tier

  - FrontEnd - written with Python Flask and using Bootstrap.
  - APITier - API App Server written in Python using Flask. Exposes an API to retrieve data from MySQL. The API App Server is also retrieving data from a Redis in memory cache that contains data from aircrafts that are currently airborne. There's a service written in Python that retrieves the Data about airborne aircrafts and pushes that data into Redis
  - DBTier - holds Aircraft registration data from the FA
  - CacheTier - Redis Cache server
  - Bastion - Used as a jumpbox
  - Cloud_Network_1 - Public Subnet
  - Cloud_Network_2 - Private Subnet

## CAS Deployment workflow

![](https://github.com/riazvm/planespotterCloud/blob/master/planespotter-master/docs/pics/casworkflow.png)


## CAS Deployment Request

### Network Allocation

![](https://github.com/riazvm/planespotterCloud/blob/master/planespotter-master/docs/pics/networkalloc.png)

### Network Provisioning

![](https://github.com/riazvm/planespotterCloud/blob/master/planespotter-master/docs/pics/networkprov.png)

### Machine Allocation

![](https://github.com/riazvm/planespotterCloud/blob/master/planespotter-master/docs/pics/machinealloc.png)

### Machine Provisioning

![](https://github.com/riazvm/planespotterCloud/blob/master/planespotter-master/docs/pics/machineprov.png)
