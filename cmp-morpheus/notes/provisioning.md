# Provisioning

Morpheus refers to itself as a infrastructure-agnostic Cloud Application Management Platform (opposed to a Cloud Management Platform).

```bash
Cloud-Management-Platform
    ├── Hypervisor / Docker Host
    │        ├── VM
    │        └── Container
    └── Bare Metal Server

Morpheus
    ├── Host
    ├── Server
    └── Instance
```

[1. Instances](#1-instances)  
[2. Apps](#2-apps)  
[3. Catalog](#3-catalog)  
[4. Jobs](#4-jobs)  
[5. Codes](#5-codes)  
[6. Labs](#6-labs)

---

### 1. Instances

An Instance in Morpheus Enterprise is a representation of a **resource** or **service**, and may involve one or more VMs or containers.

```bash
Instance
└── MongoDB Cluster
      ├── Node 1 → VM
      ├── Node 2 → VM
      ├── Node 3 → VM
      └── ...
```

In the example above, Morpheus sees MongoDB Cluster as an Instance instead of VMs/Instances.

Hence, Instance actions that can be performed to expand capacity on an Instance: adding more nodes as horizontal scaling, or more computing resources of a single node as vertical scaling.
 
**Nodes/Containers/VMs**

- An Instance can have many nodes. A **Node** is a representation of a Container or a VM.

	```bash
	Node
	├── VM
	└── Container
	```

**Hosts/Servers**

- A **Host** refers to a *Docker Host* in which a container (within an Instance) is running, or a *hypervisor* that VMs can be provisioned onto. So on, Host is where the nodes running.

	```bash
	Docker Host
		│
		├── Container A
		├── Container B
		└── Container C

	# In this case, Docker Host is considered as a host
	```

	```bash
	VMware ESXi Host
		│
		├── VM-01
		├── VM-02
		└── VM-03

	# Hypervisor/EXSi can be represented as a host by Morpheus.
	```

- A **server** is the representation of a physical or virtual server resource that Morpheus acknowledge. It could be a Host representation, a Virtual Machine, or even a Bare Metal.  

- Server and Node may be confused b/c they may point to the same VM. A Server shows a VM to Morpheus as a resource, while Node sees VM as a part of an Instance.

	```bash
											Morpheus
												│
					┌──────────────┴──────────────┐
					│                             │
			Application view             Infrastructure view
					│                             │
			Instance                       Server
					│                             │
				Node                   VM / Host / Bare Metal
					│
		VM / Container
	```

**Brownfield**  

- Morpheus can be integrated into existing environments and manage existing VMs (periodically syncing existing VMs, server record will be created and periodically updated).

---

### 2. Apps

An App is a collection of Instances linked together via application tiers.

Tiers allow the user to define separated sections of connectivity between the various Instances within an application.

```bash
App
 │
 ├── Instance
 │     │
 │     └── Node
 │            │
 │            └── VM / Container
 │
 ├── Instance
 │     └── Node
 │
 └── Instance
       └── Node
```

An App Blueprint is a user-define application structure (template) for easy deployment into various environments.

Apps can be created from Blueprints, which are made in [Library &gt; Blueprints &gt; App Blueprints]() or from Existing Apps.

---

### 3. Catalog

The Catalog presents a simplified self-service view where users can select (as shopping) and deploy Instances, Blueprints or Workflows with pre-defined configuration.

Within the Catalog, users are presented with selections based on User Role. By default, User Roles have no access to any catalog items. Thus, administrators will need to enable access to some Catalog Items.

Configuring Global Access:

- Full: Gives access to all Catalog Items

- Custom: Gives access to individually-selected items from the list below

- None: No access is given to any Catalog Items

Administrators can create and manage Catalog Items at [Library &gt; Blueprints &gt; Catalog Items]().

---

### 4. Jobs

Jobs provide the orchestration mechanism to execute **Automation Tasks** or **Workflows**.  

Jobs can be set to execute on a schedule, at one specific point in time, and/or manually.   

Job types:

- Task Job: Executes a single, specific Automation Task selected from [Library &gt; Automation &gt; Tasks]().

- Workflow Job: Triggers an entire Operational Workflow composed of a sequence of combined Tasks defined in [Library &gt; Automation &gt; Workflows]().

- Security Scan Job: Initiates security compliance and vulnerability scanning by applying pre-configured Security Packages (SCAP/STIG).

Jobs are configured in the `JOBS` tab, and the `JOB EXECUTIONS` tab contains Job execution history with result output.

---

### 5. Codes

[Provisioning &gt; Code]() provides the tools and integrations necessary to manage the Repositories, Deployments and Code Integrations sections.

The **Repositories** section contains the repositories integrated with Morpheus.

The **Deployments** section provides PaaS like capabilities when it comes to deploying applications into the newly provisioned environment.

The **Integrations** section is where Code Integrations (Git/Github Repo Integrations, Jenkins Build Service Integrations) can be created and managed.

---

### 6. Labs