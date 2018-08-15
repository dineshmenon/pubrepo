# Administering large scale PostgreSQL installations on SAP MultiCloud platform

SAP MultiCloud Platform is an open platform-as-a-service (PaaS) product that provides core platform and backing services, for building and extending cloud applications on multiple cloud infrastructure providers. SAP MultiCloud Platform presently supports AWS, OpenStack, Azure and Google Cloud Platform (GCP). Together on all infrastructures platform has approximately 8000 postgresql clusters up and running.

### Postgresql-as-a-Service is robust and intelligent enough to remain up and running

In a cluster there are two postgresql nodes running. One node runs as a primary and other node runs as a standby (also called as a hot standby). standby node is replicating primary with replication set to asynchronous mode. Primary node failure is mitigated by promoting standby node to primary. So cluster is up and running always. To detect the the failure [pgpool] is used which continusly checks the heartbeat of postgresql process. [pgpool] node is running in three nodes inorder to form consensus. Refer below figure for more understanding.

[![N|Solid](https://github.com/dbossap/dbos-performance/blob/master/clusterSetup2.png?raw=true)](https://nodesource.com/products/nsolid)

### Disaster Recovery situation is handled seamlessly with the help of Backup and Restore (B&R) and High Availability (HA)

Backup and Restore helps user to take online backup of running cluster on MultiCloud platform. The backup approach will differ for respective cloud providers. For e.g. on OpenStack we use 'tarball' approach, for other cloud providers (aws, azure and gcp) we use 'snapshot' based approach. Incase of failure/disaster situation this backup can be restored to avoid any data lose. Also at any point of time user can move the cluster to previous state(data) by restoring appropriate backup.

High availability (HA) feature helps to mitigate the failure situation. HA detects primary node failure and promote standby node to primary. Split brain is the well know and dangerous problem that could occur in this architecture (Primary-standby). This problem is avoided using STONITH operation. Failed node is killed after promoting the standby node to primary.

Client/Customer always connect to single endpoint which always points to primary node. Incase of failure situation, HA makes sure that single endpoint provided to client will always point to primary node with minimal downtime.

### Integration with [bosh] and [service-fabrik] makes Postgresql-as-a-Service a scalable and independent component on MultiCloud platform

[Bosh] is an important component of SAP MultiCloud platform. It is an open source project that offers a tool chain for release engineering, deployment & life-cycle management of large scale distributed services. All the postgresql clusters are deployed using [bosh] on MultiCloud platform. [Bosh] also helps in accessing and maintaining the cluster up and running. All the [bosh] life-cycle operations are triggered through [service-fabrik].

[Service-fabrik] is an open source component of SAP MultiCloud platform which acts as a broker between customers/clients and Postgresql-as-a-service. All the service operations are triggered through [service-fabrik]. Some operations are scheduled for e.g. scheduled backup, cluster security updates etc. Operations such as create cluster, delete cluster, update cluster, upgrade cluster will be triggered by end user customers/clients through [service-fabrik].

### Updating Postgresql-as-a-Service with new features and OS security patches is fairly easy with rolling updates (bi-weekly updates) of clusters

SAP MultiCloud platform supports rolling updates of clusters. Every postgresql cluster is updated bi-weekly with new features or bug fixes (if any). This keeps Postgresql-as-a-service improving. 
OS security patches will be applied on all clusters once in a month. With this update service get rid of security vulnerability (if any).

### SAP MultiCloud platform monitors health of every postgresql cluster

Monitoring component captures various metrics of the postgresql cluster which helps to monitor the health of the each cluster. Some of the core metrics captured are memory usage, cpu usage, disk usage etc. Also metrics such as number active connections, bulk data read, replication status, availability, failover status (if any), backup status gives detail cluster information to identify any failure.

All the metrics are captured by an agent running inside postgresql node and send those to [riemann]. [Riemann] pushes these metrics to [influxdb] where all the metrics will be stored. [Grafana] is used to show all the metrics on its dashboard. [Grafana] fetches these metrics from [influxdb] and displays it. [Grafana] gives user flexibility to select cluster, select date and time range etc. Monitoring is one of the important component SAP MultiCloud platform. Refer below figure

[![N|Solid](https://github.com/dbossap/dbos-performance/blob/master/grafana.png?raw=true)](https://nodesource.com/products/nsolid)


### Alerting module of SAP MultiCloud platform quickly identifies and notifies the failures

As a part of alerting module alert rules are configured in [riemann] on metrics like availability, backup, disk usage etc. When these metrics does not adhere to [riemann] alert rules an alert will be raised.

### Debugging is independent of postgrsql cluster availabilty

All the logs generated out of service will be pushed to ELK stack so that user can access these logs incase of any failure. So debugging is independent of service instance availability. Refer below figure

[![N|Solid](https://github.com/dbossap/dbos-performance/blob/master/kibana.png?raw=true)](https://nodesource.com/products/nsolid)

### SAP MultiCloud platform supports variety of plans with major version upgrade

Currently platform supports both 9.4 and 9.6 version of postgresql with different t-shirt sizes (plans). As per requirement, user has the flexibility to choose the plan. Plans provided will be like v9.4-small, v9.6-small, v9.6-large etc. Each plan is unique combination of disk, cpu, memory etc. 
Platform also supports major version upgrade of postgresql service. This feature easily allows user to upgrade its postgresql instance to next higher version.

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

   [aws]: <https://aws.amazon.com>
   [azure]: <https://azure.microsoft.com/en-us/>
   [gcp]: <https://cloud.google.com/>
   [openstack]: <https://www.openstack.org/>
   [bosh]: <https://github.com/joemccann/dillinger>
   [pgpool]: <https://github.com/joemccann/dillinger.git>
   [grafana]: <https://grafana.com/>
   [riemann]: <http://riemann.io/>
   [influxdb]: <https://www.influxdata.com/time-series-platform/influxdb/>
   [service-fabrik]: <https://github.com/cloudfoundry-incubator/service-fabrik-broker>
