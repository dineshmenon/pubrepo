# Securing PostgreSQL Service in a cloud environment

SAP Cloud Platform (SCP) is an open platform-as-a-service (PaaS) product that provides core services, for building and extending cloud applications on multiple cloud IAAS. SCP supports AWS, OpenStack, Azure and GCP.

>One of the core services provided by SCP is # *__PostgreSQL as a Service (PostgreSQL-as-a-Service)__*. Each PostgreSQL-as-a-Service instance consists of 5 VMs (PG-Master,PG-Standby, 3-PGPOOL VMs). PostgreSQL-as-a-Service instances of each tenant should be isolated from others. Due to the myriad of challenges and possible attack vectors in cloud services, it is imperative that PostgreSQL-as-a-Service runs securely within the platform.

Measures taken to harden the security aspects of PostgreSQL-as-a-Service:
 
### Network Security
With scaling out PostgreSQL-as-a-Service over multiple AZs, network-related aspects are important. In most of the infrastructures, an AZ maps to a subnet with a fixed block of IP addresses, spanned over a virtual network. In Azure and GCP, it is possible to have a subnet that spans over multiple AZs in a region because subnets are treated as regional resources.

##### Isolation between control plane and data plane components

- PostgreSQL-as-a-Service instances and applications that make use of postgresSQL service are setup at a networking level to reside in two *__different virtual networks__*. Each virtual network gives us complete control like selection of custom IP-address-range, creation of subnets, and configuration of route tables/network gateways.
- Within the virtual network, PostgreSQL-as-a-Service instances reside in a *__different subnet__* from control plane operators like Bosh (deployment-and-lifecycle-management-IAAS-agnostic-framework)  and Service-Fabrik (orchestrator-compoenent).

##### Leveraging infrastructure level security
 
- Whenever a PostgreSQL-as-a-Service gets launched, it gets associated with one or more *__security groups__*, wihch acts as a virtual firewall that controls the traffic to instance vms.
- DoS attacks via source *__IP spoofing__* is prevented by enabling appropriate flags in the IAAS.
- 
### Isolation among PostgreSQL-as-a-Service instances
- In order to achieve isolation among PostgreSQL-as-a-Service instances, *__iptables-manager__* is used. Whenever a PostgreSQL-as-a-Service instance gets created, it applies necessary iptable rules to each of the VMs in such a way that only those VMs will be able to communicate among themselves.
- All traffic to and from other postgresql service instances gets blocked, thus compromised and others are insulated.
- Necessary rules are applied to prevent *__ICMP__* based attacks.

### Isolation among processes in a PostgreSQL-as-a-Service instance

- Each postgreSQL instance runs as a *__non-root user__* with access enough to access the resources required for postgreSQL processes to execute, thereby relying on DAC (Discretionary-Access-Control).

- In order to provide further isolation and a mechanism for supporting access control security policies, including  mandatory access controls (MAC), *__SELinux__* is used. Here each postgres process is confined to the required set of files, port (bind and connect) and other resources by enforcing mandatory access control policies.

### Limiting resource usage at process level

In order to limit usage and to make sure that postgres processes aren't hogged for resources (CPU, file descriptors), *__getrlimit()__* and *__setrlimit()__* functions are used.

### Limiting access to system calls using seccomp

Processes in PostgreSQL-as-a-Service service instance is sandboxed such that they can only make use of required system calls. This is done using *__seccomp__*, one of the efficient ways of filtering system calls which is in turn based on BPF (Berkeley-Packet-Filter).
