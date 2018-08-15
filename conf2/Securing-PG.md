# Securing PostgreSQL Service in a cloud environment

SAP Cloud Platform is an open platform-as-a-service (PaaS) product that provides core platform and backing services, for building and extending cloud applications on multiple cloud infrastructure providers. SAP Cloud Platform presently supports AWS, OpenStack, Azure and Google Cloud Platform (GCP).

>One of the core services provided by SAP Cloud Platform is # *__PostgreSQL as a Service__*. Each PostgreSQL service instance consists of 5 VMs (1 PG Master, 1 PG Standby and 3 PGPOOL VMs). Since SAP Cloud Platform has a multi-tenant model - postgresql service instances (and associated apps) of each tenant should be isolated from others. Due to the myriad of challenges and possible attack vectors in cloud services, it is imperative that postgresql service runs securely within the platform.

Several measures taken to harden the security aspects of PostgreSQL service includes but is not limited to the following.
 
### Network Security
IAAS providers are typically hosted in multiple locations world-wide which is composed of regions. Each region is a separate geographic area with  multiple, isolated locations known as Availability Zones for achieving fault tolerance. AZs are connected to each other using low latency links and is typically employed to achieve HA and resiliency.

With scaling out postgresql service over multiple availability zones, there are network-related aspects to be addressed. In most of the infrastructures, an availability zone maps to a subnet with a fixed block of IP addresses, spanned unilaterally over a virtual network. In Azure and GCP, it is possible to have a subnet that spans over multiple AZs in a region because subnets are treated as regional resources.

##### Isolation between control plane and data plane components

- PostgreSQL service instances and applications that make use of postgresSQL service are setup at a networking level to reside in two *__different virtual networks__*. Each virtual network gives us complete control over the networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.
- Within the virtual network, postgresql service instances reside in a *__different subnet__* and control plane operators like Bosh (deployment and lifecycle management IAAS agnostic framework)  and Service Fabrik (the orchestrator compoenent) reside in different ones ensuring further isolation.
- The exposure of postgresql services also gets limited by using a private subnet with no internet access directly.

##### Leveraging infrastructure level security
 
- Whenever a posgresql instance gets launched, it gets associated with one or more *__security groups__*. A security group acts as a virtual firewall that controls the traffic for one or more instances. The rules associated with the group enables the service to allow traffic to or from its associated vms. 
- Since new rules can be applied at runtime to all of the postgreSQL service instances, runtime attacks from compromised sources are neutralized by applying appropriate rules.
- Denial of Service (DoS) attacks via source *__IP spoofing__* is also prevented by enabling appropriate flags in the IAAS level.

### Isolation among postgreSQL service instances
- In orde to achieve isolation among postgeresql instances, a component called *__iptables-manager__* is used. Whenever a postgreSQL service instance gets created, this component applies necessary iptable rules to each of the VMs in the given service instance in such a way that only those VMs will be able to communicate among themselves. 
- All traffic to and from other postgresql service instances gets blocked, thereby making sure that even if any service instance get compromised, other service instances are not effected.
- Similarly, necessary rules are applied to prevent *__ICMP__* based attacks.

### Isolation among processes in a service instance
Every VM in a postgreSQL service instance runs postgresql related processes as well other supporting and required processes in the operating system. In order to make sure that postgreSQL process is sufficiently isolated from others, following security hardening measures are taken :

- Each postgreSQL instance runs as a *__non-root user__* with access enough to access the resources required for postgreSQL processes to execute, thereby relying on DAC (Discretionary Access Control) mechanism. So even if the postgreSQL process(es) gets compromised (demo to showcase this point using reverse shell hack), access will be limited to postgreSQL resources .

- In order to provide further isolation and a mechanism for supporting access control security policies, including  mandatory access controls (MAC), *__SELinux__* is used. Here each postgres process is confined to the required set of files, port (bind and connect) and other resources by enforcing mandatory access control policies. Thus even if the user is compromised, only the process with the right labelling would be able to access the files (resources).

### Limiting resource usage at process level

In order to make sure that all processes including postgreSQL process in the service instance VM gets the required share of resources (CPU, file descriptors), *__getrlimit()__* and *__setrlimit()__* functions are used. This guarantees that the usage of the process is limited and postgresql processes will not get hogged by other processes in the system.


### Limiting access to system calls using seccomp

Each of the processes in the postgreSQL service instance is sandboxed so that they can only make use of the required system calls. This is done using *__seccomp__*, one of the efficient ways of filtering system calls which is in turn based on BPF (Berkeley Packet Filter, a programmable packet filtering and classification system that runs within the kernel). Thus even if a process is compromised, seccomp restricts the usage of system calls to the bare minimum required.
