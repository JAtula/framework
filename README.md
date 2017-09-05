# Cloud provider agnostic
## Framework for microservice web development with mostly OSS tools
The idea of this document is to answer the following architectural and design parameters about a complete framework for web app development. The app would live in the cloud, but the design is cloud provider agnostic (as much as possible) so there's no provider lock-in. All the tools are such, that I've used before in production or demo purposes.

* The service is web-based that end users use with any modern browser.
* The service has to be cloud provider agnostic.
* The app follows the rules of the 12-factor app (as much as possible). Read the 12-factor app manifesto [here] (https://12factor.net/).
* The backend has many different DBs that have to be backed up automatically.
* The app has to have an ACL controlled API layer.
* For development reasons, many versions of the app need to be available for the stake holders.
* The app has to scale automatically.
* Central logging has to exist.
* Central metrics has to exist.
* The philosophy of infrastructure-as-code needs to be applied to underlying hardware.

## Backend

The services will be containerized and orchestrated with **Kubernetes** (K8S). We'll use a provider-specific tool for the infrastructure templating, so for example with Azure, we'd use **ARM templates**, or **Heat** for **OpenStack**. The container runtime will be orchestrated with **Ansible**, as it is an easy-to-pick-up-and-go tool for infrastructure management that devs and ops alike can use. Ansible is an agentless model for runtime management, so if "drift control" is must, we'll use **SaltStack**. 

The workload nodes will be scheduled as an autoscaling group, so that load will never exceed processing capacity.

**GitLab** will be used for version control as it offers Kubernetes plug-in and CI/CD out-of-the-box. 

Different versions of the app will be run on the same K8S cluster or different K8S clusters,according to business requirements.

Metrics will be collected on workload nodes using a **Prometheus** and **Grafana** stack. Alerting is done either with Prometheus alert manager or through Grafana. 

All DBs will be clustered according to best practices given by the product and/or the provider to achieve redundancy in case of failure for the "hot" data. Backups will taken according to a retention policy. I'll recommend using an elastic NoSQL or SQL database that all the three big cloud providers offer. So for: 

#### Azure
* Cosmos DB. It offers an SLA of .99 and support for relational and non-relational DBs.

* *Azure SQL for MSSQL*

#### AWS
* Amazon RDS for relational and, 
* Dynamo DB for non-relational DBs. The former has a .95 SLA with multi-AZ instance deployments, and the latter .99 as S3 is used to house the data.

#### GCP
* Cloud Spanner for relational (SLA of .99) and, 
* Cloud Datastorage for NoSQL DBs (SLA of .95).

Good DB options with OSS or proprietary products would be for example:

* PostgreSQL 
* CockroachDB
* MSSQL
* MariaDB

## Frontend

We'll use **HAProxy** for load balancing as it has a wide set ways to control ACL. So for example, we could limit access based on:

* Host headers.
* Source IP.
* Address path.

The downside of HAProxy is that it doesn't "hot reload" it's configurations, so will use runtime management for reloading configuration when needed. 

For mobile access, well use **OAuth2** and a Valet Key Pattern (read more [here](https://docs.microsoft.com/en-us/azure/architecture/patterns/valet-key)) for issuing ephemeral tokens for the client to consume resources. All the major cloud providers offer OAuth2 to integrate into your app, but you could use OSS like [**Gluu**](https://www.gluu.org/).

Application logs and metrics will be collected with **Elasticsearch**, **Logstash** and **Kibana**. Logstash will be configured as a central collection point for **Filebeat** which we'll we use for sending stdout to Logstash.

## Testing

I haven't touched testing at all, and even though it's not part of the framework as given by the requirements, testing is a critical part of any development and should be part in every stage. As a "hack" we could use **Robot Framework** to do acceptance testing and **Gatling** for load and performance testing. 

## An example drawing deployed to GCP

![Drawing](https://github.com/JAtula/framework/blob/master/diagram.png?raw=true "draw.io drawing")