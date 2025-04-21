# Cilium Certified Associate

_This exam is an online, proctored, multiple-choice exam._

## Resources

* <https://github.com/isovalent/CCA-Study-Guide>
* <https://cilium.io/blog/2018/04/24/cilium-security-for-age-of-microservices/>

## Topics

<details>
  <summary>Architecture (20%)</summary>

Cilium is an open source CNI compatible networking and scurity layer for Kubernetes. Thanks to BGP, a powerful kernel extensibility mechanism inside of Linux, Cilium rethinks the Linux networking.

In-kernel networking brings:

* Performance - workload is already traversing local kernel.
* Transparency - data are sent using TCP/IP, no need to change applications
* Security

BPF: Framework for running custom logic at various hook points in the kernel.

BPF programs: The logic, JIT compiled.

Service centric identity and API awareness, with Cilium identity is extracted from the contianer orchestrator and embedded in each network request.

Cilium removes the need of a sidecar proxy, per Pod proxy between microservices, the Istio or Envoy way of doing this suboptimally by traversing the TCP/IP stack. BPF brings "sockmap".

## Components

![alt text](image.png)

### Cilium

* Agent, runs on each node. Accepts config via Kubernetes. Listens on events from orchestration systems _such as_ Kubernetes. Manages eBPG programs which the Linux kernel uses to control all network access.
* Debug Client, `cilium-dbg`. Interacts with the REST API of the Cilium agent running on the same node. Inspects state and status of the local agent.
* Operator, responsible for managing duries in the cluster which should logically be handled once for the entire cluster, rather than once for each node. _Not in the critical path for functionality on the network layer._. If there's a failure in the Operator this might happen:
  * Delays in IPAM and thus scheduling of new workloads.
  * Failure to update **kvstore** heartbeat key.
* CNI Plugin, invoked when a pod is scheduled or terminated on a node. Interacts with the Cilium API of the node.

### Hubble

* Server, runs on each node at retrieves the eBPF visibility from Cilium.
* Relay, standalone component which is aware of all running Hubble servers and offers cluster-wide visibility.
* Client CLI, retrieve flow events via the gRPC API of the relay.
* GUI

### eBPF

eBPF is a Linux kernel bytecode interpreter. In-kernel verifier ensures that eBPF programs are safe to run and a JIT compiler converts the bytecode to CPU architecture.

### Data Store

State propagation between agents are done via:

* CRD, the default
* Key-Value store, an optimization. etcd as the only supported one. Imagine having a seperate etcd for this and not use the same as the cluster.

### eBPF Data Path

The Linux kernel supports a set of BPF hooks in the networking stack that can be used to run BPF programs. The Cilium **datapath** uses these hooks to load BPF programs to create _higher level networking contructs_.

* XDP is at the earliest point possible in the networking driver. Ideal for running filtering programs that drop malicious or unexpected traffic, DDOS protection mechanism!
* Traffic Control, TC, ingress/egress. Runs **after** the networking stack has done initial processing of the packet. Runs before L3. Containers typically use a virtual device called a veth pair which acts like a virtual wire connecting the container to the host. TC ingress hook is added on the host side.
* Socket Operations, this hook is attached to a specific cgroup and runs on TCP events.
* Socket send/recv runs on every send operation performed by a TCP socket

The combination of the hooks above and virtual interfaces, `cilium_host` and `cilium_net`, an optional overlay interface `cilium_vxlan`, Linux user space crypto support and a userspace proxy (Envoy) Cilium creates the following networking objects:

* Prefilter
* Endpoint Policy, this is the primary object in the Cilium datapath responsible for mapping packets to identities and enforcing L3 and L5 policies
* Service
* L3 Encryption
* Socket Layer Enforcement
* L7 Policy

#### The life of a packet

There's three different scenarios:

* Endpoint to Endpoint
* Egress from Endpoint
* Ingress to Endpoint

#### eBPF Maps

All BPF maps are created with a upper capacity limits.

#### IPtables usage

Depending on the Linux kernel version used, the eBPF datapath can implrement a varying feature set fully in eBPF. If required capabilities are not available the functionality provided using a leagacy iptables implementation.

### IPAM

Responsible for the allocation and management of IP addresses used by network endpoints (containers and others). Dont change the IPAM mode of an existing cluster, this may cause persistent disruption of connectivity for existing workloads.

There's different modes:

* Cluster Scope (default) - assigns per-node PodCIDRs to each node and allocates IPs using a host-scope allocator on each node. Similar to Kubernetes Host Scope mode. Cluster scope uses the `CiliumNode` instead of the `v1.Node`.
* Kubernetes Host Scope - `ipam: kubernetes` flag.
* Multi-Pool
* Azure IPAM
* Azure Delegated IPAM
* AWS ENI
* GKE
* CRD-backed - extendable interface to control the IP address management via a Kubernetes CRD. Allows to delegate IPAM to external operators. Operator watches `ciliumnodes.cilium.io`.

### Labels

Whenever something needs to be described, addressed or selected, it is done based on labels:

* Endpoint are assinged labels

A label is a pair of strings consisting of a `key` and `value`.

A label can be derived from various sources, for example an endpoint will derive the labels associated to the container by the local container **runtime** as well as the labels associated with **pod** as provided by Kubernetes.

Example of a `CiliumEndpoint`:

```
apiVersion: cilium.io/v2
kind: CiliumEndpoint
status:
  encryption: {}
  external-identifiers:
    cni-attachment-id: e2ca9c7df3e13300b82ec641463736ef91daa50dba96a40477bdb688d46c0e81:eth0
    container-id: e2ca9c7df3e13300b82ec641463736ef91daa50dba96a40477bdb688d46c0e81
    k8s-namespace: default
    k8s-pod-name: andqvi-temp-shell
    pod-name: default/andqvi-temp-shell
  id: 2473
  identity:
    id: 80868
    labels:
    - k8s:io.cilium.k8s.namespace.labels.field.cattle.io/projectId=p-l2b4c
    - k8s:io.cilium.k8s.namespace.labels.kubernetes.io/metadata.name=default
    - k8s:io.cilium.k8s.policy.cluster=k8s-devstage01
    - k8s:io.cilium.k8s.policy.serviceaccount=default
    - k8s:io.kubernetes.pod.namespace=default
    - k8s:run=andqvi-temp-shell
    - k8s:topology.kubernetes.io/region=DC07
    - k8s:topology.kubernetes.io/zone=dc07
  networking:
    addressing:
    - ipv4: 10.169.3.73
    node: 10.171.13.142
  state: ready
```

Supported label sources:

* `container:` from the local container runtime
* `k8s` derived from Kubernetes
* `reserved` for special reserved labels
* `unspec` with an unspecified source

The endpoint ID is an internal id that Cilium assigns to all endpoints on a cluster node. The endpoint ID is unique within the context of an individual cluster node.

All endpoints are assigned an identity, the identity is what is used **to enforce basic connectivity between endpoints**. Equvialent to L3 enforcement. An identity is **identified by Labels and is given a cluster wide unique identifier**.

Examples on special identities:
* `reserved:unknown` - identity could not be derived.
* `reserved:host` - Local host.
* `reserved:world` - Any network endpoint **outside** of the cluster.

Well known identities are a set of identities that Cilium is aware of automatically. E.g. `kube-dns`, `core-dns` etc.



</details>

<details>
  <summary>Network Policy (18%)</summary>

* Interpret Cilium Network Polices and Intent
* Understand Cilium's Identity-based Network Security Model
* Policy Enforcement Modes
* Policy Rule Structure
* Kubernetes Network Policies versus Cilium Network Policies

</details>

<details>
  <summary>Service Mesh (16%)</summary>

* Know How to use Ingress or Gateway API for Ingress Routing
* Service Mesh Use Cases
* Understand the Benefits of Gateway API over Ingress
* Encrypting Traffic in Transit with Cilium
* Sidecar-based versus Sidecarless Architectures

</details>

<details>
  <summary>Network Observability (10%)</summary>

* Understand the Observability Capabilities of Hubble
* Enabling Layer 7 Protocol Visibility
* Know How to Use Hubble from the Command Line or the Hubble UI

</details>

<details>
  <summary>Installation and Configuration (10%)</summary>

* Know How to Use Cilium CLI to Query and Modify the Configuration
* Using Cilium CLI to Install Cilium, Run Connectivity Tests, and Monitor its Status

</details>

<details>
  <summary>Cluster Mesh (10%)</summary>

* Understand the Benefits of Cluster Mesh for Multi-cluster Connectivity
* Achieve Service Discovery and Load Balancing Across Clusters with Cluster Mesh

</details>

<details>
  <summary>eBPF (10%)</summary>

* Understand the Role of eBPF in Cilium
* eBPF Key Benefits
* eBPF-based Platforms versus IPtables-based Platforms

</details>

<details>
  <summary>BGP and External Networking (6%)</summary>

* Egress Connectivity Requirements
* Understand Options to Connect Cilium-managed Clusters with External Networks

</details>
