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
* *Application*: Code performing a function, which is deployed on the platform.

## 1. Layer 7 Security

First-line, first-class security is implemented at the application
layer. Additional layers of security such as firewalls, VPNs, VPCs, etc.
are viewed as convenience layers providing protection from attacks
of scale (such as DDOS) which can take down an application providing
adequate layer 7 security.

Provided by *end-users*.

## 2. Single Permission Profile

All end-users, from ops to devops to devs, have the same
access to cluster nodes. Allowing *operators* privileged access will
result in creating processes requiring their intervention.

Related to:

* No SSH Access

Provided by *operators*

## 3. No Host Access

No user ever has access to any underlying node in the cluster.
This means starting from pre-baked images, and having some means of
accessing retired disks/memory-dumps for post-mortem diagnostics.
Additionally, adequate diagnostics/telemetry data are collected
to ensure the need for post-mortem data extraction is minimal.

Using EBS-backed instances on AWS and persisting the volumes on instance
termination can assist with post-mortem investigations.

Related to:

* Immutable Filesystems
* Single Permission Profile

Depends on:

* Transient Storage

Provided by *operators*

## 4. Transient Storage

Storage is ephemeral, which means data must be
replicated, runtime-local, or streamed off of the application. Nodes are
capable of coming and going at will, with applications being rebalanced
along with any required storage.

This is probably the most technically challenging item on the list, and
one easy solution is to assume that no data is ever persistent, and require
applications to be architected to always use a backing data-store as a
resource.

Ultimately, the goal is to allow the application to declare replicated,
persisted data
volumes, which can be live-migrated across hosts during a rebalancing.

Related to:

* Immutable Filesystems

Provided by:

* Flocker
* ConvergeIO
* GlusterFS?
* NFS?
* Avoiding local storage

## 5. Immutable Filesystems

All cluster nodes' underlying filesystems are immutable. This requires
adhering to the transient storage model. With immutable filesystems, hosts
can essentially be 'reset' or 'wiped clean' by a restart, which means
maintenance efforts are minimized.

Depends on:

* Transient Storage

Provided by:

* CoreOS
* RancherOS

## 6. Uniform Nodes

Cluster nodes are considered identical, regardless of their underlying
architecture. Nodes may be of different sizes or speeds, which can be
taken into account by the platform tooling. Any node requiring special
maintenance or properties is to be avoided. No special snowflakes.

Perhaps not realistic if running behemoth jobs or DBs (redis) that want
to expand to fill available memory.

Provided by *operators*.

## 7. Uniform Applications

No application is considered 'special', 'unique' or in any way requires
specific thought as to its placement or resource consumption. If an app
does require specific attention, it will be further deconstructed or
refactored so as to no longer require special care.

Provided by *developers*.

## 8. Initiator-based Application Placement

No end-user of the cloud needs to know or care about
where an application they've deployed has landed. Containers
deployed from a development environment (laptop, devbox, etc.) will
automatically
interoperate with all other containers deployed from the same source.

The naive architecture in this case is a single large cluster of nodes,
in which the configuration of the application at runtime sets it up to
connect to the appropriate other nodes. For enhanced security or other
compliance purposes, this may actually exist as one cluster per
environment, but this should be transparent to the end-user.

This is **sole** means by which code can
go from a build to running in a shared environment.

Note: This might be convention more than factor. Label-based autogen environments.

Provided by *operators*

## 9. Cluster and Container Metrics Only
No one, from ops to end-users, needs to care about host-level metrics.
Concerns are solely around performance of a particular application,
and the performance of the cluster as a whole. Applications are designed
appropriately such that resource contention or scaling can be handled
by expanding horizontally, by automated rebalancing, or a combination of the
two.

Provided by:

* Mesos/Marathon
* Kubernetes
* Rancher

## 10. Real-Time Aggregated Logging
Log streams are an integral part of the platform, and
route all logs in such a way that end-users can access real-time log
streams, as well as persisted log storage. Stream access should be
available for end-user-managed aggregation and log mining.

There is no confusion about what logs go where, or how to find the logs for
a given application.

Provided by:

* ?

