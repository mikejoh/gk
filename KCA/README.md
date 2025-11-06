# Kyverno Certified Associate

_This exam is an online, proctored, multiple-choice exam._

## Resources

* <https://kyverno.io/docs/>

## Topics

<details>
  <summary>Fundamentals of Kyverno (18%)</summary>

* Kyverno Policies & Rules
* YAML Manifests
* Admission Controllers
* OCI Images

Kyverno is **a cloud native policy engine**. Originally built for Kubernetes but can now also be used outside of Kubernetes.

Kyverno allows platform engineers to automate security, compliance and best practices validation and deliver secure self-service to application teams.

## How Kyverno works

### Installing

The simplest way of installing `kyverno` is to use: `kubectl create -f <https://github.com/kyverno/kyverno/releases/latest/download/install.yaml>`

The following components are installed in the `kyverno` namespace:

* Kyverno Admission Controller
* Kyverno Background Controller
* Kyverno Cleanup Controller
* Kyverno Report Controller



### Admission Controls

Kyverno runs as a **dynamic admission controller** in a Kubernetes cluster. Kyverno receives **validating** and **mutating** admission **webhook HTTP callbacks** from the Kubernetes API server.

Mutating policies can be written as overlays (like Kustomize) OR as **JSON patches**. Validating policies als use an overlay style syntax, with support for pattern matching and conditional (if-then-else) processing.

Policy **enforcement** is captured using **Kubernetes** events. For requests that are either allowed or existed prior to introduction of a Kyverno policy then Kyverno creates **Policy Reports** in the cluster which contain a running list of resources matched by a policy, their status and more.

High-level architecture of Kyverno:

![alt text](image.png)

The **Webhook** is the server that handles incoming **AdmissionReview** requests from the Kubernetes API server and sends them to the **Engine** for processing.

The **Cert Renewer** is responsible for watching and renewing the certificates, stored as Kubernetes Secrets, needed by the webhook.

The **Background Controller** handles all generate and mutate-existing policies by reconciling `UpdateRequest`, which is a intermediary resource. And the **Report Controller** handle creaion and reconciliation of Policy Reports.

Kyverno supports **high-availability**, which means that controllers that are selected for installation are configured to run multile replicas.

### Policies and Rules

Kyverno can match resources using the resource's fields such as:

* kind
* name
* label
* selectors

</details>

<details>
  <summary>Installation, Configuration and Upgrades (18%)</summary>

* Helm-based Installation and Configuration
* Kyverno Custom Resource Definitions (CRDs)
* Controller Configuration with Flags
* Configuring Kyverno RBAC, roles, and permissions
* High Availability Installations
* Upgrading Kyverno

</details>

<details>
  <summary>Kyverno CLI (12%)</summary>

* apply
* test
* jp
* Installing Kyverno CLI

</details>

<details>
  <summary>Applying Policies (10%)</summary>

* Applying Policy in Cluster
* Resource Selection
* Common Policy Settings for Kyverno Rules

</details>

<details>
  <summary>Writing Policies (32%)</summary>

* Validation Rules
* Preconditions
* Background Scans
* Mutation Rules
* Generation Rules
* VerifyImage Rules
* Variables & API Calls in Policies
* JSON Patches
* Autogen Rules
* Cleanup Policies
* Common Expression Language (CEL)

</details>

<details>
  <summary>Policy Management (10%)</summary>
* Policy Reports
* PolicyExceptions
* Kyverno Metrics

</details>
