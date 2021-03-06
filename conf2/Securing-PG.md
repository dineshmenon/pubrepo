SAP Cloud Platform (SCP) is an open platform-as-a-service (PaaS) product that provides core services, for building and extending cloud applications on multiple cloud IAAS. SCP supports AWS, OpenStack, Azure and GCP.

[![N|Solid](https://github.com/dineshmenon/pubrepo/blob/master/resc/secure-pg/SAP_Cloud_Platform.png?raw=true)](https://nodesource.com/products/nsolid)

    One of the core services provided by SCP is PostgreSQL as a Service (PostgreSQL-as-a-Service). Each PostgreSQL-as-a-Service instance(cluster) consists of 5 VMs (PG-Master, PG-Standby, 3-PGPOOL VMs). 

[![N|Solid](https://raw.githubusercontent.com/dbossap/dbos-performance/master/postgresql-Cluster.png?raw=true)](https://nodesource.com/products/nsolid)

##### PostgreSQL-as-a-Service instances of each tenant should be isolated from others. Due to the myriad of challenges and possible attack vectors in cloud services, follwing measures are taken to run it securely within the platform.

### Network Security
With scaling out PostgreSQL-as-a-Service over multiple AZs, network-related aspects are important. In most of the infrastructures, an AZ maps to a subnet with a fixed block of IP addresses, spanned over a virtual network. In Azure and GCP, it is possible to have a subnet that spans over multiple AZs in a region because subnets are treated as regional resources.

[![N|Solid](https://github.com/dineshmenon/pubrepo/blob/master/resc/secure-pg/az-subnet.png?raw=true)](https://nodesource.com/products/nsolid)

### Isolation between control and data plane

- PostgreSQL-as-a-Service instances and applications that make use of postgresSQL service are setup at a networking level to reside in two *__different virtual networks__*. Each virtual network gives complete control on selection of custom IP-address-range, creation of subnets, and configuration of route tables/network gateways.

- Within the virtual-network, PostgreSQL-as-a-Service instances reside in a *__different subnet__* from control plane operators like Bosh (deployment-and-lifecycle-management-IAAS-agnostic-framework)  and Service-Fabrik (orchestrator-compoenent).

### Leveraging infrastructure level security
 
- Whenever a PostgreSQL-as-a-Service gets launched, it gets associated with one or more __security groups__, wihch acts as a virtual firewall that controls the traffic to instance vms.
- DoS attacks via source __IP spoofing__ is prevented by enabling appropriate flags in the IAAS.
### Isolation among PostgreSQL-as-a-Service instances
- In order to achieve isolation among PostgreSQL-as-a-Service instances, __iptables-manager__ is used which applies necessary iptable rules to each of the VMs in such that only instance VMs will be able to communicate among themselves.
- All traffic to and from other postgresql service instances gets blocked, thus compromised and others are insulated.
- Necessary rules are applied to prevent __ICMP__ based attacks.

### Isolation among processes in a PostgreSQL-as-a-Service instance

- Each PostgreSQL-as-a-Service instance runs as a __non-root user__ with privileges enough to access the resources required for postgreSQL processes to execute, thereby relying on DAC (Discretionary-Access-Control).

- To provide further isolation and a mechanism for supporting access control security policies, including  mandatory access controls (MAC), __SELinux__ is used. Each postgres process is confined to the required set of files, port (bind and connect) and other resources by enforcing mandatory access control policies.

### Limiting resource usage at process level

    In order to limit usage and to make sure that postgres processes aren't hogged for resources (CPU, file-descriptors), getrlimit() and setrlimit() functions are used.

### Limiting access to system calls using seccomp

    Processes in PostgreSQL-as-a-Service service instance is sandboxed such that they can only make use of required system calls. This is done using *__seccomp__*, one of the efficient ways of filtering system calls based on BPF (Berkeley-Packet-Filter).