# Twelve-Factor Platform

Definitions:

* *User*: Any employee or participant in the organization implementing the
  platform.
* *Operator*: A member of the team responsible for implementing, maintaining
  and trouble-shooting the platform.
* *End-user*: Anyone pushing code into the cloud. Typically a developer, but
  could also be ops and devops if they're deploying code and not actually
  building infrastructure.
* *Migrate*: The act of moving an application from one underlying
  cluster node to another.
* *Live-migrate*: The act of *migrating* an application while persisting
  the application's state, data, and connections.
* *Balance (a cluster)*: The act of (live-)migrating applications to
  maximize usage of a cluster's resources.

1. Layer 7 Security
  First-line, first-class security should be implemented at the application
  layer. Additional layers of security such as firewalls, VPNs, VPCs, etc.
  should be viewed as convenience layers providing protection from attacks
  of scale (such as DDOS) which would take down an application providing
  adequate layer 7 security.

  Provided by end-users.

1. Single Permission Profile
  All end-users, from ops to devops to devs, should have the same
  access to cluster nodes. Allowing operations privileged access will
  result in creating process requiring their intervention.

  Related to:

  * No SSH Access

  Provided by policy.

1. No SSH Access
  No users should ever have access to any underlying node in the cluster.
  This means starting from pre-baked images, and having some means of
  accessing retired disks/memory-dumps for post-mortem diagnostics.
  Additionally, adequate diagnostics/telemetry data will need to be collected
  to ensure the need for post-mortem data extraction is minimal.

  Related to:

  * Immutable Filesystems

  Depends on:

  * Transient Storage

  Provided by policy.

1. Transient Storage
  Storage should be assumed to be transient always, which means it must be
  either replicated, support hot backups, or ideally both. Nodes should be
  capable of coming and going at will, with applications being rebalanced
  along with any required storage.

  This is probably the most technically challenging item on the list, and
  one easy solution is to assume that no data is ever persistent, and require
  applications to be architected to always use a backing data-store as a
  resource.

  Ultimately, the goal is to allow the application no declare persisted data
  volumes, which can be live-migrated across hosts during a rebalancing.

  Related to:

  * Immutable Filesystems

  Provided by:

  * Flocker
  * ConvergeIO
  * GlusterFS?
  * NFS?

1. Immutable Filesystems
  All cluster nodes' underlying filesystems should be immutable. This requires
  adhering to the transient storage model. With immutable filesystems, hosts
  can essentially be 'reset' or 'wiped clean' by a restart, which means
  maintenance efforts are minimized.

  Depends on:

  * Transient Storage

  Provided by:

  * CoreOS
  * RancherOS

1. Uniform Nodes
  Cluster nodes are considered identical, regardless of their underlying
  architecture. Nodes may be of different sizes or speeds, which can be
  taken into account by the platform tooling. Any node requiring special
  maintenance or properties is
  to be avoided.

  Perhaps not realistic if running behemoth jobs or DBs (redis) that want
  to expand to fill available memory.

  Provided by operators.

1. Uniform Applications
  No application should be considered 'special', 'unique' or in any way require
  specific thought as to its placement or resource consumption. If an app
  does require specific attention, it should be further decomposed or
  refactored so as to no longer require special care.

  Provided by developers.

1. Initiator-based Application Placement
  No end-user of the cloud should need
  or care to know where an application they've deployed has landed. Containers
  deployed from a dev environment (laptop, devbox, etc.) should automatically
  interoperate with all other containers deployed from the same source.

  The naive architecture in this case is a single large cluster of nodes,
  in which the configuration of the application at runtime sets it up to
  connect to the appropriate other nodes. For enhanced security or other
  compliance purposes, this may actually look like one cluster per
  environment, but this should be transparent to the end-user.

  Might be convention more than factor. Label-based autogen environments.

  Should be the *sole* means by which code can
  go from a build to running in a shared environment.

  Provided by:

  * ?

1. Cluster and Container Metrics Only
  No one, from ops to end-users, should need to care about host-level metrics.
  Concerns should be solely around performance of a particular application,
  and the performance of the cluster as a whole. Applications should be scaled
  appropriately such that resource competition or scaling can be handled
  horizontally, by automated relocation, or a combination of the two.

  Provided by:

  * Mesos/Marathon
  * Kubernetes
  * Rancher

1. Real-Time Aggregated Logging
  Log streams should be an integrated part of the platform, and should
  route all logs in such a way that end-users can access real-time log
  streams, as well as persisted log storage. Stream access should be
  available for end-user-managed aggregation and log mining.

  Provided by:

  * ?

1. Minimal Image Size

