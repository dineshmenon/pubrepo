# High Availability (HA) for Postgresql as a Service in SAP Multi-Cloud Platform

SAP Cloud Platform is an open platform-as-a-service (PaaS) product that provides core platform and backing services, for building and extending cloud applications on multiple cloud infrastructure providers. SAP Cloud Platform presently supports AWS, OpenStack, Azure and Google Cloud Platform (GCP).

>One of the core services provided by SAP Cloud Platform is # *__PostgreSQL as a Service__*. Each PostgreSQL service instance consists of 5 VMs (1 PG Master, 1 PG Standby and 3 PGPOOL VMs).

### PostgreSQL HA cluster using pgpool-II

Each PostgreSQL service instance consists of 5 VMs (1 PG Master, 1 PG Standby and 3 PGPOOL VMs). Standby node replicates the data being written to primary node in an asynchronous way. Primary node failure is mitigated by promoting standby node to primary, thereby acheiving high availability. To detect the the failover scenario of postgres master node, [pgpool] is used. pgpool monitors the backends (postgresql service vms) configured in the cluster and triggers the failover command when it identifies a failure.

[![N|Solid](https://github.com/dbossap/dbos-performance/blob/master/clusterSetup2.png?raw=true)](https://nodesource.com/products/nsolid)

Once the failover is performed, STONITH (Shoot The Other Node In The Head) is performed. Once the operation is performed, old master is resurrected (vm is recreated) and necessary binaries are loaded for the node to function appropriately. The old master then joins the cluster in standby mode.

Bosh (deployment and lifecycle management IAAS agnostic framework) is responsible for "resurrecting" postgreSQL vms when it becomes unresponsive. It is also responsible for applying required binaries to all VMs in the postgreSQL service instance VMs.

### Importance of Single Endpoint for PostgreSQL Service in Cloud

Each postgreSQL service instance consists of two postgreSQL VMs which could be in either master or standby mode. At any point of time, applications may connect to the service instance and it is imperative that applications should have a mechanism to connect to the service instance via a single endpoint/ single ip. 

It is also worth noting that only jdbc driver supports multiple IPs in the connection string and is intelligent enough to make the distinction of modes of postgreSQL instances. In the case of drivers in other languages, this is no longer true. Each of these drivers expect the ip of the master node at any point of time. Thus the importance of single endpoint in a cloud environment is of high importance.

### Approach to achieve HA & Single Endpoint in Azure & GCP

In the case of Azure Cloud, a standard internal load balancer is associated with each service instance. The client ip (single ip) is associated with the load balancer as the frontend ip address. Both the postgresvms are made as a part of backend pool of the load balancer. A forwarding rule is configured to forward the requests to the backend pool. 
A health check probe runs in both of the vms - it returns 200 OK from the master node nad 500 ISR from the standby node. This probe is associated with the load balancer as the health check probe.

Thus at any point in time, load balancer forwards the traffic to the *__healthy__* node only, in this case, master node in the postgreSQL service instance.

Various steps involved in this implementation is depicted below. 

[![N|Solid](https://github.com/dineshmenon/pubrepo/blob/master/resc/ha/Azure-Implementation.png?raw=true)]()


### Approach to achieve HA & Single Endpoint in AWS


### Approach to achieve HA & Single Endpoint in Openstack


### Approach to achieve HA & Single Endpoint in Openstack
One of the core services in SAP 




