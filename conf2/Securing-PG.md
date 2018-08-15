# Securing PostgreSQL Service in a cloud environment

SAP Cloud Platform (SCP) is an open platform-as-a-service (PaaS) product that provides core platform and backing services, for building and extending cloud applications on multiple cloud infrastructure providers. SCP presently supports AWS, OpenStack, Azure and GCP.

>One of the core services provided by SCP is # *__PostgreSQL as a Service (PostgreSQL-as-a-Service)__*. Each PostgreSQL-as-a-Service instance consists of 5 VMs (PG-Master,PG-Standby and 3 PGPOOL VMs). Since SCP has a multi-tenant model, PostgreSQL-as-a-Service instances of each tenant should be isolated from others. Due to the myriad of challenges and possible attack vectors in cloud services, it is imperative that PostgreSQL-as-a-Service runs securely within the platform.

Measures taken to harden the security aspects of PostgreSQL-as-a-Service:
 
### Network Security
With scaling out PostgreSQL-as-a-Service over multiple AZs, there are network-related aspects to be addressed. In most of the infrastructures, an AZ maps to a subnet with a fixed block of IP addresses, spanned unilaterally over a virtual network. In Azure and GCP, it is possible to have a subnet that spans over multiple AZs in a region because subnets are treated as regional resources.

##### Isolation between control plane and data plane components

- PostgreSQL-as-a-Service instances and applications that make use of postgresSQL service are setup at a networking level to reside in two *__different virtual networks__*. Each virtual network gives us complete control over the networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.
- Within the virtual network, postgresql service instances reside in a *__different subnet__* from control plane operators like Bosh (deployment-and-lifecycle-management-IAAS-agnostic-framework)  and Service Fabrik (orchestrator-compoenent).

##### Leveraging infrastructure level security
 
- Whenever a PostgreSQL-as-a-Service gets launched, it gets associated with one or more *__security groups__*. A security group acts as a virtual firewall that controls the traffic for one or more instances. The rules associated with the group enables the service to allow traffic to or from its associated vms.
- DoS attacks via source *__IP spoofing__* is prevented by enabling appropriate flags in the IAAS.
- 
### Isolation among postgreSQL service instances
- In order to achieve isolation among PostgreSQL-as-a-Service instances, *__iptables-manager__* is used. Whenever a PostgreSQL-as-a-Service instance gets created, it applies necessary iptable rules to each of the VMs in such a way that only those VMs will be able to communicate among themselves.
- All traffic to and from other postgresql service instances gets blocked, thus compromised and others are insulated.
- Necessary rules are applied to prevent *__ICMP__* based attacks.

### Isolation among processes in a service instance
Every VM in a postgreSQL service instance runs postgresql related processes as well other supporting and required processes in the operating system. In order to make sure that postgreSQL process is sufficiently isolated from others, following security hardening measures are taken :

- Each postgreSQL instance runs as a *__non-root user__* with access enough to access the resources required for postgreSQL processes to execute, thereby relying on DAC (Discretionary Access Control) mechanism. So even if the postgreSQL process(es) gets compromised (demo to showcase this point using reverse shell hack), access will be limited to postgreSQL resources .

- In order to provide further isolation and a mechanism for supporting access control security policies, including  mandatory access controls (MAC), *__SELinux__* is used. Here each postgres process is confined to the required set of files, port (bind and connect) and other resources by enforcing mandatory access control policies. Thus even if the user is compromised, only the process with the right labelling would be able to access the files (resources).

### Limiting resource usage at process level

In order to make sure that all processes including postgreSQL process in the service instance VM gets the required share of resources (CPU, file descriptors), *__getrlimit()__* and *__setrlimit()__* functions are used. This guarantees that the usage of the process is limited and postgresql processes will not get hogged by other processes in the system.


### Limiting access to system calls using seccomp

Each of the processes in the postgreSQL service instance is sandboxed so that they can only make use of the required system calls. This is done using *__seccomp__*, one of the efficient ways of filtering system calls which is in turn based on BPF (Berkeley Packet Filter, a programmable packet filtering and classification system that runs within the kernel). Thus even if a process is compromised, seccomp restricts the usage of system calls to the bare minimum required.
