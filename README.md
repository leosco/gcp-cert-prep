# Google Cloud Platform Certification Prep

## Associate Cloud Engineer

### Links

- https://www.udemy.com/course/latest-gcp-ace-google-associate-cloud-engineer-practice-exams-tests/
- https://quizlet.com/434285600/gcp-assoc-engineeer-flash-card-set-1-flash-cards/
- https://quizlet.com/434286726/gcp-assoc-engineeer-flash-card-set-2-flash-cards/
- https://quizlet.com/434287399/gcp-assoc-engineeer-flash-card-set-3-flash-cards/
- https://quizlet.com/434288229/gcp-assoc-engineeer-flash-card-set-4-flash-cards/
- https://cloud.google.com/learn/certification/cloud-engineer

### Disorganized, incomplete, maybe not entirely accurate notes

**Cloud Storage**

- Storage classes
  - Standard (no min duration, no read fees, 99.99% SLA)
  - Nearline (30 day duration, read fees, 99.9% SLA)
  - Coldline (90 day duration, read fees, 99.9% SLA)
  - Archive (365 day duration, read fees, 99.9% SLA)
- Object lifecycle policy can be used to adjust the class and retention of objects in a bucket
- Making a GCS bucket public requires setting Storage Object Viewer Role for that bucket to allUsers (special public group)
- Signed URLs can be created for buckets to expose the contents over the internet, and can be given a time-to-live
- Storage buckets are ideal for hosting static content and unstructured data of different file types
- GCS object lifecycle policies are not sequential, I.e do not offset the ages based on linear progression of time. However, you can optimize cost by considering the cumulative age of an object. If an object is in Standard storage for 30 days, there’s no point in moving it to Nearline for extended period of time because you’ve already satisfied the 30 day requirement. The next cheapest class, Coldline, is optimal because it requires objects to be stored at least 90 days at lower cost.
- If you have a massive amount of data to migrate into GCP, Google offers a transfer appliance service that ships you hard drives to copy your data into and physically send back to Google to load into your project's buckets

**Compute Engine**

- Existing compute subnet CIDRs are expandable, no need for additional subnets or peering if space is the only concern
- VPCs have dynamic routing between all members of the network, firewall rules control ingress/egress of devices on the network
- Compute VMs can be tagged to be handled by different firewall rules
- Preemptible VMs are much cheaper than standard Compute VMs b/c they can be shutdown by Google at any time
- Shielded VMs have advanced intrusion prevention against rootkits, bootkits
- Cloud VPN allows compute VPCs to be exposed securely to on-prem networks, with options to configure static and dynamic routes
- Compute VMs can have snapshot schedules to backup their disk and set retention policy on those backups
- VM vCPUs as well as disk size/type and other configurations have an impact on performance, see https://cloud.google.com/compute/docs/disks/performance
- GSuite login does not work for Windows VMs and instead requires bespoke credentials and connecting via RDP “gcloud compute reset-windows-password”
- Users cannot SSH into compute VMs by default, the instances must have oslogin enabled and the users must have the compute.osLogin role
- GCP best practices never recommend distributing SSH keys to enable compute access
- Project-level SSH keys are ideal for granting project-wide SSH access to Linux-based Compute VMs in the project

**BigQuery and Looker**

- Data Studio is a self-service BI solution, now folded into Looker
- BigQuery supports external tables for Storage and Bigtable
- Log sinks can aggregate logs from multiple projects into BigQuery datasets
- BigQuery requires the jobUser role to run queries, dataViewer only allows for viewing canned reports
- BigQuery charges based on the bytes processed per job, use `bq –dry-run` to see how many bytes would be processed by a query. Don’t manually calculate charges, use the GCP Pricing Calculator to get more accurate pricing

**Kubernetes Engine**

- GKE can run on GPU instances via NVidia Tesla and can be managed by GCP via node auto-provisioning
- VPC peering is the best way to expose a GKE cluster’s internal load balanced service to VMs in another VPC
- GKE shielded nodes are run on Shielded Compute VMs, it also verifies the integrity of a node’s identity cryptographically via kubelet certificate, validating that the node is a GCP VM, validating that the node is part of the cluster’s MiG
- GKE Autopilot vs Standard mode: Autopilot is fully managed by Google, including node infra, scaling, upgrades, baseline security and networking; standard you manage individual nodes, nodepools, and Kubernetes version + configurations
- To increase number of nodes in a nodepool with gcloud, use `gcloud container clusters resize <name> --node-pool <pool_name> --num-nodes <number_of_nodes>`
- Whenever GKE workloads have significantly different resource requirements (GPU, CPU, memory, etc), best practice is to group those by nodepool so that workloads can be scheduled onto nodes best equipped to serve them
- Deployments are the typical way to deploy containerized workloads with the option to run multiple replicas
- DaemonSets maintain pods on every node, i.e. a deployment that will have a workload on every node in the cluster
- StatefulSets are like Deployments but have consistent network identifiers, persistent storage, ordered CRUD
- Services enable service discovery within and without the cluster, depending on configuration; it decouples consumers of a Pod's endpoints from the Pod's IP address

**IAM and Billing**

- Service accounts can be consumed via keyfile or Federated Workload Identity which maps third-party identities to service accounts
- Basic roles give very broad, unpredictable permissions that should generally be avoided
- Predefined roles are scoped to particular GCP products and resources
- A "principal" is anything that can be assigned IAM roles and can consume GCP APIs
- IAM policies are scoped to a project, so are service accounts; if you need to assign a service account in a different project permissions to consume resources in your project, the IAM policy is on your project referencing the service account from the other via fully qualified name
- Distributing service account keys is bad practice and instead we should use Cloud Identity where possible
- Cloud Identity allows you to configure third-party idP for SSO generally
- You can migrate your Active Directory to GCP using Google Cloud Directory Sync
- roles/browser allows full hierarchy view of GCP org
- roles/iam.securityReviewer is more permissive than Viewer apparently
- Custom roles allow you to combine IAM permissions, which have varying support
  - SUPPORTED means the permission is fully supported to work in a custom role
  - TESTING means the permission is supported in a custom role but may not be production-ready
  - NOT_SUPPORTED means the permission is not supported in a custom role
  - Custom roles can be marked with a release stage/channel such as ALPHA, BETA, GA, DEPRECATED
- You can easily redirect projects from one billing account to another
- You can manage IAM policies for specific principals via `gcloud projects add-policy-binding`
- Inspect a project's IAM policies with `gcloud projects get-iam-policy`
- Always use the Pricing Calculator for estimations, do not pick answers that have you doing your own math or provisioning unneeded resources to see what happens
- To move a project to a billing account, you need to be both admin on the Billing Account as well as Project Owner

**Databases**

- Cloud Spanner, CloudSQL, and BigQuery are all relational
  - CloudSQL is for when you need a true SQL-flavored database, supports multi-regional read replica but writes are limited to a single regional instance
  - Spanner is a general-purpose first-party Google product that's globally distributed SQLlike database with low latency and high throughput
  - BigQuery is a SQLlike data warehouse for analytics, reporting, and BI; it has first class integration with Looker for reporting, and can source data from various products (Storage, Logging, Dataflow, Bigtable, Spanner)
- Bigtable and Firestore are non-relational
  - Bigtable is the noSQL alternative to Spanner, a super fast global database for unstructured data with multi-primary replication across regions
  - Firestore is a document-based noSQL comparable to MongoDB, ideal for application data and integration with App Engine

**Serverless**

- Pub/Sub supports pushing messages to Cloud Run instances as long as the service account has Cloud Run Invoker role
- For richer network and other configuration options in App Engine, use Flexible typed environments, i.e. connecting to an on-prem database via Cloud VPN
- Cloud Run is ideal for fully managed containerized workloads and applications that require traffic splitting
- Cloud Run can always run "cold" or a minimum number of active instances can be configured to run "hot"
- Cloud Functions are lighter weight code snippets that are run on-demand via triggers, such as when a GCS bucket gets a new file
- App Engine has two ways of traffic splitting: IP or Cookie based; cookie is more precise because it identifies the end user’s browser regardless of source IP. For something other than a browser-based application, IP is more appropriate
- App Engine CLI does not support automatic rollback (wow); probably you need to catch the error in your smoketest and repromote the previous known version

**Logging and Monitoring**

- Cloud Monitoring allows you to setup workspaces to observe the performance metrics of resources across multiple projects
- Cloud Logging allows you to set arbitrary labels on existing logs for easier aggregation
- Auditing is tricky, but generally Admin Activity is logged by default; other types of logs such as Data Access for GCS are enabled at the resource level
- Log sinks can ship logs to various destinations, including buckets for long-term storage, pub/sub for serverless actions, BigQuery for analytics
- Monitoring and logging both support the configuration of alert criteria, including by projecting custom fields via regex, and delivery via email and other services

**Misc**

- You can launch third-party managed services from GCP marketplace images w/ one click, i.e MongoDB Atlas, Hadoop, CloudAMQP, and more
- Deployment Manager is a first-party GCP IaC tool
- Shutting down GCP project causes all resources to turn off and then are deleted after a 30 day period. This action requires roles/project.owner
- Generally avoid exposing resources or routing traffic through the internet unless an explicit requirement
- Cloud Foundation Toolkit features Google-recommended templates for quickly bootstrapping environments
- “Partner network” implies over the public internet
- Provision on-demand vs overprovisioning; while performing a migration, always increment capacity and leverage budgets, quotas, and autoscaling features
- Capacity != quota; capacity is what is actually provisioned, quota is a capacity limit
- Automate everything, avoid answers that involve manual intervention unless explicitly specified as a requirement of the question, i.e. auditors needing view access to resources or reports
- Minimal cost + effort means writing no additional code, i.e. “lift-and-shift”, really you’re choosing the lowest cost option with the least effort, which may be far from the lowest cost option if you put any work into it
- Cloud shell provisions 5GB of free $HOME directory space and puts ~/bin in your path automatically
