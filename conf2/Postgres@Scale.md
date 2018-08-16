SAP Cloud Platform (SCP) is an open platform-as-a-service (PaaS) product that provides core services, for building and extending cloud applications on multiple cloud IAASs. SCP supports AWS, OpenStack, Azure and GCP.

    One of the core services provided by SCP is PostgreSQL as a Service (PostgreSQL-as-a-Service). Each PostgreSQL-as-a-Service instance consists of 5 VMs - Postgres-Master, Postgres-Standby and 3-PGPOOL VMs. Data is replicated asynchronously from Postgres-Master to Postgres-Standby.

### Postgresql-as-a-Service - Robust & Highly Available

[pgpool] VMs continuously monitor the health of postgres VMs. In case of any failures, [pgpool] triggers the promotion of Postgres-Standby to Postgres-Master. Failover process is comprised of **STONITH** operation for auto-correction.

[![N|Solid](https://github.com/dbossap/dbos-performance/blob/master/clusterSetup2.png?raw=true)](https://nodesource.com/products/nsolid)

### Multiple-Plans-to-support-all-use-cases

Multiple plans of PostgreSQL-as-a-Service are made available based on #cpu_cores, memory and disk size associated with instance VMs. 

[![N|Solid](https://github.com/dineshmenon/pubrepo/blob/master/resc/pg-scale/pg_plans.png?raw=true)](https://nodesource.com/products/nsolid)

SCP manages more than **10000 PostgreSQL-as-a-Service instances** across multiple IAASs.

### Disaster-Recovery/Data-Protection-using-Backup-and-Restore

PostgreSQL-as-a-Service instance provides point-in-time-recovery**(PITR)** using WAL archiving. A base backup along with WAL files are archived on cloud storage. Snapshot based base-backup is taken in AWS/Azure/GCP. In Openstack, base backup is taken by copying and compressing data directory. 

PostgreSQL-as-a-Service remains available during base backup. Recovery process involves restoration of data directory from base-backup and replay of WAL logs to a desired "reocvery-time-objective".

We use [**Bosh**](https://bosh.io/docs/) for managing the deployment and life cycle management of PostgreSQL in a large scale. Bosh is an open source project that offers a tool chain for release engineering, deployment & life-cycle management of large scale distributed services. The lifecycle management operations are triggered through another open source component from SAP MultiCloud platform called [**Service-Fabirk (SF)**](https://github.com/cloudfoundry-incubator/service-fabrik-broker). Some of the operation triggered through SF component are feature update of the cluster, update the cluster to a different T-Shirt size plan, schedule backup for all the clusters.

Every cluster goes through a **bi-weekly rolling update** for introducing new features. When the primary server gets updated as part of this rolling update, the standby server is promoted to primary. The rolling update involves near **zero down time**, and so the service is always available for use. 

A **monitoring agent** runs in every PostgreSQL VM to report its health metrics. These metrics include VM related information like CPU, memory and disk usage, and database related information like availability of the service, replication, number of active connections etc. The monitoring agent collects this information and reports it to [**Riemann**](http://riemann.io/) which is a monitoring component. Riemann processes the metrics and stores it in [**Influxdb**](https://www.influxdata.com/time-series-platform/influxdb/) which is a time series database, and also sends an alerts if some undesired state is reported, like primary server is not available, replication is down, disk size crossed a certain threshold limit. [**Grafana**](https://grafana.com/), another monitoring component, shows all these time series data from Influxdb in a dashboard.
![N|Solid](https://github.com/AbhijitGharami/postgres-conf/blob/master/PostgreSQL_HA_HighLevel.png?raw=true)



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
