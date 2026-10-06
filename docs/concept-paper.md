# Wazuh-Siem2 -- Concept Paper

**Version:** 0.1\
**Status:** Draft / Architecture Baseline\
**Project:** Wazuh-Siem2\
**Current phase:** Milestone 1 -- Architecture & Platform Foundation\
**Target environment:** Kubernetes\
**Target users:** Small and Medium-sized Enterprises (SMEs / KKV)

------------------------------------------------------------------------

## 1. Executive Summary

Wazuh-Siem2 is a project to design and build a structured, documented
and multi-tenant-ready Security Information and Event Management (SIEM)
platform based on Wazuh.

The objective is not simply to deploy Wazuh, but to create a reusable
SIEM service platform that can support multiple independent customer
environments while providing consistent log collection, security event
processing, detection, alerting, reporting and operational processes.

The initial target market is small and medium-sized enterprises (SMEs).
The platform therefore focuses on infrastructure and log sources
commonly found in SME environments, including:

-   Windows 10/11 endpoints
-   Windows Server
-   Active Directory
-   Linux servers
-   MikroTik
-   FortiGate
-   pfSense / OPNsense
-   Proxmox
-   VMware
-   Docker
-   IIS
-   databases
-   NAS systems
-   other syslog-capable network and infrastructure devices

The platform is designed as **multi-tenant-ready from the beginning**,
even if the first development and proof-of-concept environments contain
only one or two tenants.

Wazuh remains the core SIEM engine, while **syslog-ng is introduced as a
dedicated collection layer** between customer infrastructure and the
central Wazuh platform.

The long-term runtime environment of the platform is Kubernetes.

The project deliberately excludes general-purpose big-data analytics.
Its scope is security monitoring and SIEM services.

------------------------------------------------------------------------

## 2. Project Motivation

Deploying a SIEM for a single organization is relatively
straightforward.

Operating a SIEM as a reusable service for multiple independent
customers introduces additional architectural requirements.

The platform must be able to answer, for every collected security event:

-   Which customer generated the event?
-   Which customer site generated it?
-   Which device generated it?
-   What type of source generated it?
-   Through which transport channel did it arrive?
-   How should it be stored?
-   Which detection rules apply?
-   Who is allowed to access it?
-   How long should it be retained?

These questions influence the architecture from the collection layer
through processing, indexing, dashboards and long-term operations.

For this reason, tenant awareness cannot be added only at the final
production stage.

It must be part of the architecture from the beginning.

------------------------------------------------------------------------

## 3. Project Goals

The main goal of Wazuh-Siem2 is to create a reusable SIEM platform that
can later be operated as a service for multiple SME customers.

The platform should provide:

-   centralized security log collection
-   security event processing
-   threat detection
-   alerting
-   event normalization
-   customer and source identification
-   tenant separation
-   reporting
-   configurable retention
-   auditable configuration management
-   scalable operation
-   Kubernetes-based deployment

The architecture should allow the same platform design to be used
during:

1.  local development
2.  proof-of-concept deployments
3.  company development environments
4.  production environments

Environment-specific configuration should be separated from the core
platform design wherever possible.

------------------------------------------------------------------------

## 4. Non-Goals

The following areas are currently outside the scope of Wazuh-Siem2:

-   general-purpose big-data analytics
-   business intelligence
-   enterprise data lake functionality
-   generic log analytics unrelated to security
-   large-scale machine-learning analytics
-   general application observability platform functionality

These capabilities may integrate with the platform in the future, but
they are not architectural goals of the current project.

Wazuh-Siem2 should remain focused on SIEM and security monitoring.

------------------------------------------------------------------------

## 5. Target Environment

The initial target environment is a typical SME infrastructure.

A customer may have one or multiple sites containing combinations of:

### Endpoint systems

-   Windows 10
-   Windows 11
-   Windows Server
-   Linux

### Identity infrastructure

-   Active Directory
-   Windows authentication services

### Network infrastructure

-   MikroTik
-   FortiGate
-   pfSense
-   OPNsense
-   switches
-   routers
-   VPN gateways
-   other syslog-capable network devices

### Virtualization and container infrastructure

-   Proxmox
-   VMware
-   Docker

### Application infrastructure

-   IIS
-   application servers
-   database servers
-   NAS systems

The source list is intentionally focused on technologies commonly found
in smaller and medium-sized organizations rather than attempting to
support every possible enterprise product from the beginning.

------------------------------------------------------------------------

## 6. Architecture Principles

The Wazuh-Siem2 architecture follows several core principles.

### 6.1 Multi-tenancy from the beginning

Tenant awareness is part of the architecture even during early
development.

Every event entering the platform should eventually be attributable to a
specific tenant and source context.

### 6.2 Clear separation of responsibilities

Collection, transport, SIEM processing, indexing and visualization
should remain logically separated.

This allows individual layers to evolve without redesigning the entire
platform.

### 6.3 Infrastructure as Code

Platform configuration should be version controlled whenever possible.

This includes:

-   Kubernetes manifests
-   syslog-ng configuration
-   Wazuh configuration
-   custom decoders
-   custom rules
-   deployment configuration
-   test configuration

### 6.4 Git-based development

Changes should be introduced through small and understandable commits.

This provides:

-   traceability
-   auditability
-   rollback capability
-   reproducible environments
-   easier code review

### 6.5 Kubernetes as the target platform

The initial proof-of-concept environment may use Docker or a small local
Kubernetes cluster, but architectural decisions should avoid unnecessary
dependencies on a single-host deployment model.

### 6.6 Security by design

Customer separation, transport security, authentication, authorization
and auditability must be architectural concerns rather than later
additions.

------------------------------------------------------------------------

## 7. High-Level Architecture

The baseline logical log flow is:

``` text
Customer Log Sources
        │
        │
        │ Wazuh Agent / Syslog / VPN / TLS
        │
        ▼
+-------------------------------+
|      syslog-ng Collection     |
|             Layer             |
|                               |
|  - tenant identification      |
|  - source identification      |
|  - metadata enrichment        |
|  - filtering                  |
|  - buffering                  |
|  - routing                    |
+---------------+---------------+
                │
                ▼
+-------------------------------+
|         Wazuh Manager         |
|                               |
|  - decoding                   |
|  - rules                      |
|  - security analysis          |
|  - alert generation           |
+---------------+---------------+
                │
                ▼
+-------------------------------+
|         Wazuh Indexer         |
|                               |
|  - indexing                   |
|  - search                     |
|  - retention                  |
+---------------+---------------+
                │
                ▼
+-------------------------------+
|        Wazuh Dashboard        |
|                               |
|  - visualization              |
|  - investigation              |
|  - reporting                  |
+-------------------------------+
```

This represents the conceptual architecture.

Specific log sources may use different transport paths, but the tenant
and source context must remain identifiable throughout the processing
chain.

------------------------------------------------------------------------

## 8. Why syslog-ng Is Part of the Architecture

syslog-ng is not considered an optional utility in Wazuh-Siem2.

It is a dedicated architectural component.

Wazuh provides the SIEM processing layer, but a multi-customer service
also requires a flexible collection and transport layer.

The syslog-ng layer provides a controlled point for:

-   receiving syslog traffic
-   identifying incoming customer channels
-   source classification
-   metadata enrichment
-   filtering
-   routing
-   buffering
-   protocol handling
-   transport isolation
-   forwarding events toward Wazuh

This separation also reduces the need to make Wazuh responsible for all
transport-specific logic.

Conceptually:

``` text
Transport and collection responsibility
                │
                ▼
            syslog-ng

Security analysis responsibility
                │
                ▼
              Wazuh
```

This distinction is an important design principle of Wazuh-Siem2.

------------------------------------------------------------------------

## 9. Tenant Model

The platform must maintain sufficient metadata to identify the context
of every event.

The initial conceptual hierarchy is:

``` text
Tenant
  │
  ├── Site
  │     │
  │     ├── Device
  │     │      │
  │     │      └── Log Source
  │     │
  │     └── Device
  │
  └── Site
```

A future event metadata model is expected to contain information similar
to:

``` text
tenant_id
site_id
device_id
source_type
transport
```

Additional metadata may later include:

``` text
environment
device_role
source_vendor
source_product
source_version
collector
ingest_timestamp
```

The exact schema will be defined during Milestone 1.

The key requirement is that tenant identification must not depend solely
on free-text log content.

Where possible, tenant identity should originate from a trusted
communication or configuration context.

Examples include:

-   dedicated VPN connection
-   dedicated TLS certificate
-   dedicated syslog listener
-   Wazuh agent registration
-   predefined source mapping

------------------------------------------------------------------------

## 10. Multi-Tenant Communication Model

Customers should connect to the SIEM platform through controlled and
identifiable communication channels.

Conceptually:

``` text
Customer A ── Secure Log Transport ──┐
                                     │
Customer B ── Secure Log Transport ──┼──> Collection Layer
                                     │
Customer C ── Secure Log Transport ──┘
```

The channels should remain logically separated even if they eventually
share physical infrastructure.

Possible mechanisms include:

-   VPN
-   TLS
-   mutual TLS
-   customer-specific certificates
-   dedicated listeners
-   network segmentation
-   source allowlists

The final transport model will be defined as part of the network and
security design.

------------------------------------------------------------------------

## 11. Log Collection Models

Not all log sources should be collected using the same mechanism.

The platform therefore supports several collection patterns.

### Wazuh Agent

Preferred where an agent can provide deeper endpoint visibility.

Typical sources:

-   Windows endpoints
-   Windows Server
-   Linux servers
-   Active Directory servers

### Syslog

Preferred for infrastructure devices with native syslog support.

Typical sources:

-   MikroTik
-   FortiGate
-   pfSense
-   OPNsense
-   switches
-   routers
-   NAS devices
-   hypervisors

### Application and service logs

Some applications may require:

-   file monitoring
-   agent-based collection
-   syslog forwarding
-   dedicated integration

Examples include:

-   IIS
-   databases
-   Docker
-   application servers

The supported source matrix will document the preferred collection
mechanism for each source type.

------------------------------------------------------------------------

## 12. Windows Security Monitoring

Windows environments are considered first-class targets of the platform.

The initial Windows monitoring scope includes:

-   Windows Event Log
-   authentication events
-   Active Directory events
-   Sysmon
-   PowerShell logging
-   Windows Defender events
-   account management
-   privilege changes
-   service changes
-   suspicious process execution

Windows endpoint and Active Directory visibility are expected to provide
a significant part of the security value for SME customers.

------------------------------------------------------------------------

## 13. Detection Model

Collecting logs alone does not provide sufficient SIEM value.

The platform must convert collected events into actionable security
information.

Detection capabilities will be developed through:

-   Wazuh built-in decoders
-   custom decoders
-   Wazuh built-in rules
-   custom rules
-   event normalization
-   source-specific parsing
-   MITRE ATT&CK mapping

Initial SME-focused use cases are expected to include:

-   brute-force authentication attempts
-   repeated failed logins
-   suspicious PowerShell activity
-   privilege escalation indicators
-   suspicious account creation
-   endpoint security events
-   firewall security events
-   network attack patterns
-   administrative configuration changes

Detection content will be tested with reproducible test data wherever
possible.

------------------------------------------------------------------------

## 14. Testing Strategy

Testing is a core part of the platform rather than an activity performed
only before production deployment.

The project should gradually introduce:

-   sample log libraries
-   synthetic log generators
-   decoder tests
-   rule tests
-   regression tests
-   source integration tests
-   transport failure tests
-   buffering tests
-   Kubernetes failure tests

A particularly important test category is tenant isolation.

The platform must be tested to verify that:

``` text
Tenant A data != visible to Tenant B
```

Tenant separation tests should cover the complete path from ingestion to
indexing and visualization.

------------------------------------------------------------------------

## 15. Kubernetes Target Architecture

Kubernetes is the target runtime environment of Wazuh-Siem2.

The purpose of Kubernetes is not simply to containerize Wazuh.

It should provide an operational foundation for:

-   repeatable deployment
-   configuration management
-   service recovery
-   persistent storage
-   secret management
-   certificate management
-   controlled upgrades
-   monitoring
-   scaling
-   environment separation

The architecture must distinguish between stateless and stateful
components.

Persistent data must be stored outside the lifecycle of individual pods.

Typical persistent requirements include:

-   indexed security events
-   configuration state
-   certificates
-   required Wazuh state
-   backup data

Detailed storage design is part of Milestone 1 and Milestone 4.

------------------------------------------------------------------------

## 16. Development-to-Production Model

The platform is expected to evolve through multiple environments.

The initial development model is:

``` text
Developer
    │
    ▼
Git Repository
    │
    ▼
Home Kubernetes
Development Environment
```

This environment allows architecture and deployment concepts to be
tested without depending on company infrastructure.

As the project matures:

``` text
Developer
    │
    ▼
Git
    │
    ├── Home Kubernetes
    │      Development / POC
    │
    └── Company Kubernetes
           Integration / Production
```

The goal is to keep the core platform definition portable.

Environment-specific differences should be represented as configuration
rather than independent implementations.

Examples include:

-   storage classes
-   DNS names
-   ingress configuration
-   resource limits
-   certificates
-   secrets
-   external addresses

This enables the home development platform and future company platform
to use the same architectural baseline.

------------------------------------------------------------------------

## 17. Git and Configuration Management

Wazuh-Siem2 follows a Git-based development model.

Changes should be introduced using small, understandable commits.

Example commit sequence:

``` text
Initial Wazuh-Siem2 project structure
Add Milestone 1 documentation structure
Add platform requirements baseline
Add reference architecture baseline
Add tenant metadata model
Add syslog-ng collection layer
Add Windows Event Log source
Add MikroTik log source
Add custom Wazuh rules
Add Kubernetes deployment
```

The repository should eventually contain both documentation and
deployable configuration.

Proposed structure:

``` text
wazuh-siem2/
├── README.md
│
├── docs/
│   ├── concept-paper.md
│   ├── architecture.md
│   ├── requirements.md
│   ├── log-flow.md
│   ├── tenant-model.md
│   ├── network-design.md
│   ├── supported-log-sources.md
│   ├── security-model.md
│   └── diagrams/
│
├── kubernetes/
│   ├── base/
│   └── overlays/
│       ├── home/
│       └── company/
│
├── syslog-ng/
│
├── wazuh/
│   ├── config/
│   ├── decoders/
│   └── rules/
│
├── tests/
├── scripts/
└── .gitignore
```

The exact structure may evolve during implementation.

------------------------------------------------------------------------

## 18. Project Milestones

### Milestone 0 -- Development Environment

**Goal:** establish a working development and proof-of-concept baseline.

Status: substantially complete.

Completed work includes:

-   Wazuh single-node Docker POC
-   external syslog ingestion
-   Git repository
-   GitHub repository
-   separation from upstream Wazuh repository
-   initial project structure
-   initial project commits

### Milestone 1 -- Architecture & Platform Foundation

**Current milestone**

Goal: define the professional and architectural baseline before
significant implementation work begins.

Main tasks:

-   requirements
-   SME reference architecture
-   syslog-ng architecture
-   tenant model
-   tenant metadata standard
-   network separation
-   VPN/TLS model
-   log flow definition
-   Windows/Linux collection model
-   network device integration model
-   Kubernetes target architecture
-   storage principles
-   retention principles
-   backup principles
-   security model
-   supported log source matrix
-   architecture diagrams

Expected documentation:

``` text
docs/
├── architecture.md
├── requirements.md
├── log-flow.md
├── tenant-model.md
├── network-design.md
├── supported-log-sources.md
├── security-model.md
└── diagrams/
```

### Milestone 2 -- SME Infrastructure Log Sources

Goal: integrate and validate common SME log sources.

Initial targets:

-   Windows 10/11
-   Windows Server
-   Active Directory
-   Sysmon
-   PowerShell
-   Windows Defender
-   Linux
-   MikroTik
-   FortiGate
-   pfSense / OPNsense
-   Proxmox
-   Docker
-   IIS
-   databases
-   NAS

All integrations should use the tenant metadata model defined during
Milestone 1.

### Milestone 3 -- Detection, Parsing & Testing

Goal: convert collected logs into meaningful SIEM detections.

Main areas:

-   custom Wazuh decoders
-   custom Wazuh rules
-   normalization
-   MITRE ATT&CK mapping
-   SME-specific detection use cases
-   log generators
-   automated regression testing
-   tenant isolation testing
-   syslog-ng buffering tests
-   transport outage tests

### Milestone 4 -- Kubernetes Platform

Goal: move from the initial POC environment to a scalable Kubernetes
platform.

Main areas:

-   Wazuh Kubernetes deployment
-   syslog-ng Kubernetes architecture
-   persistent volumes
-   secret management
-   certificate management
-   backup and restore
-   scaling
-   monitoring
-   pod failure testing
-   node failure testing
-   Git-based configuration management

### Milestone 5 -- Multi-Tenant Production Platform

Goal: make the tenant-ready architecture production capable.

Main areas:

-   full tenant isolation
-   RBAC
-   tenant-specific dashboards
-   index policies
-   retention policies
-   customer onboarding
-   certificate lifecycle
-   audit trail
-   backup policy
-   SLA processes
-   operational procedures
-   monitoring
-   capacity planning
-   customer/service-level metrics

Multi-tenancy does not begin at this milestone.

It becomes production-ready at this milestone.

------------------------------------------------------------------------

## 19. Security Principles

The detailed security model will be developed separately, but several
principles already apply.

### Customer data must be identifiable

Every event must have reliable customer context.

### Customer data must be isolated

One customer must not gain access to another customer's data.

### Transport must be protected

Untrusted networks should use secure transport mechanisms such as VPN
and/or TLS.

### Credentials must not be stored in Git

Passwords, private keys and other secrets must be managed separately.

### Configuration changes must be traceable

Security-relevant configuration should be version controlled and
auditable.

### Least privilege

Components and users should receive only the permissions required for
their role.

------------------------------------------------------------------------

## 20. Storage, Retention and Backup

Security event storage can become one of the main resource consumers of
a SIEM platform.

The design must therefore consider:

-   event volume per tenant
-   retention period
-   index lifecycle
-   storage capacity
-   backup requirements
-   restore requirements
-   customer-specific retention policies

The platform should eventually support different retention profiles for
different customers or service levels.

The exact storage architecture and retention values are not defined in
Concept Paper v0.1.

They will be defined during the architecture phase based on expected
event volumes and operational requirements.

------------------------------------------------------------------------

## 21. Observability and Operations

The SIEM platform itself must also be monitored.

Future platform monitoring should include:

-   component availability
-   ingestion rate
-   event processing rate
-   queue status
-   syslog-ng buffers
-   dropped events
-   indexing performance
-   storage utilization
-   Kubernetes pod health
-   Kubernetes node health
-   certificate expiration
-   backup status

A SIEM platform that silently stops collecting events represents a
security risk.

Collection health is therefore part of the security model.

------------------------------------------------------------------------

## 22. Architecture Decisions Already Established

The following decisions form the current Wazuh-Siem2 baseline:

  Area                       Decision
  -------------------------- -----------------------------------------
  SIEM engine                Wazuh
  Collection layer           syslog-ng
  Primary target             SME / KKV infrastructure
  Architecture               Multi-tenant-ready from the beginning
  Endpoint priority          Windows and Linux
  Network device priority    MikroTik, FortiGate, pfSense / OPNsense
  Target runtime             Kubernetes
  Configuration management   Git
  Development approach       Milestone-based
  Change model               Small, auditable commits
  Big-data analytics         Outside project scope

These decisions should be considered baseline assumptions unless an
Architecture Decision Record (ADR) explicitly changes them.

------------------------------------------------------------------------

## 23. Open Architecture Questions

Concept Paper v0.1 intentionally leaves several implementation decisions
open.

These should be resolved during Milestone 1.

Examples include:

-   exact tenant metadata schema
-   where tenant metadata is injected
-   how tenant metadata survives the complete processing pipeline
-   syslog-ng deployment topology
-   customer-specific listener strategy
-   VPN topology
-   TLS / mutual TLS strategy
-   certificate lifecycle
-   Wazuh agent enrollment model
-   index separation strategy
-   tenant-specific index strategy
-   dashboard isolation model
-   RBAC model
-   Kubernetes storage technology
-   storage sizing model
-   retention tiers
-   backup architecture
-   disaster recovery requirements
-   high-availability requirements
-   syslog-ng buffering strategy
-   Wazuh Manager scaling model
-   Indexer scaling model
-   configuration promotion from development to production

These are architecture tasks, not assumptions that should be hidden
inside implementation.

------------------------------------------------------------------------

## 24. Documentation Model

The Concept Paper describes the overall project direction.

Detailed decisions belong in dedicated documents.

``` text
concept-paper.md
       │
       ├── requirements.md
       │
       ├── architecture.md
       │
       ├── tenant-model.md
       │
       ├── log-flow.md
       │
       ├── network-design.md
       │
       ├── security-model.md
       │
       ├── supported-log-sources.md
       │
       └── diagrams/
```

This keeps the Concept Paper readable while allowing the architecture
documents to become technically detailed.

Major architecture decisions may later be captured as Architecture
Decision Records (ADRs).

------------------------------------------------------------------------

## 25. Next Steps

The immediate project phase is:

**Milestone 1 -- Architecture & Platform Foundation**

The recommended documentation sequence is:

1.  establish the documentation structure
2.  define platform requirements
3.  define the reference architecture
4.  define the tenant model and metadata standard
5.  define log flows
6.  define network and secure transport architecture
7.  define the security model
8.  define the supported log source matrix
9.  create architecture diagrams
10. review and freeze the first architecture baseline

After this baseline is reviewed, implementation work can proceed with a
clear technical contract.

------------------------------------------------------------------------

## 26. Conclusion

Wazuh-Siem2 is not intended to be merely a Wazuh installation.

The objective is to create a structured, repeatable and operationally
manageable SIEM service platform for SME environments.

The most important architectural principle is that future requirements
such as multi-tenancy, secure customer connectivity and Kubernetes
operation are considered from the beginning rather than retrofitted
after the proof of concept.

The current architecture is based on three major ideas:

``` text
Wazuh
    Security analysis and SIEM engine

syslog-ng
    Collection, transport and routing layer

Kubernetes
    Long-term operational platform
```

Together with a Git-based development and configuration model, these
components provide the foundation for evolving the project from a local
development environment into a production multi-customer SIEM service.

------------------------------------------------------------------------

**Document version:** 0.1\
**Document status:** Draft\
**Next review:** Milestone 1 Architecture Baseline
