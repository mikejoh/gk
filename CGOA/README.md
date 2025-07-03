# Certified GitOps Associate

_This exam is an online, proctored, multiple-choice exam._

## Resources

* <https://training.linuxfoundation.org/certification/certified-gitops-associate-cgoa/>
* <https://github.com/otkd/CGOA-Study-Guide>

## Topics

<details>
  <summary>GitOps Terminology</summary>

* Continuous - Continues to happen, not necessarily in instantaneous. Ensures that your system remains in the desired state consistently.
* Declarative Description - A config that describes the desired operating state of a system without specifying procedures how that state will be achieved. Focus on the desired outcome rather than implementation details.
* Desired State - The Final configuration or state that you want your system to achieve.
* State Drift - Inconsistency or variance between the desired state and the actual state of the system. Identifying and addressing state drift helps maintain system **reliability** and **consistency**.
* State Reconciliation - Align the actual system state with the desired state. Reconiliation ensures that your system remain consistenc according to the defined configurations.
* GitOps Managed Software System - Software systems that ar emanaged according to GitOps principles. Leveraging GitOps principles ensures relaiability, scalability and consistency.
* State Store - Centralized repository or storage that serves as the single source of truth for your cluster's configuration and desired state.
* Feedback Loop - A process to gather and incorporating feedback from various sources before executing the reconciliation process.
* Rollback - Methods: `git revert` to undo. Automatic rollback using tools like ArgoCD. It's a safty net.

_GitOps is practice of managing infrastructure and apps with Git as the source of truth._

GitOps is a software **delivery philosophy** that treats source code managment systems like Git as the single source of truth for applications.

</details>

<details>
  <summary>GitOps Principles</summary>

* Declarative - You systems configuration and desired state should be clearly expressed in a declarative manner. This makes it easer to understand and manage what your system should look like.
* Versioned and Immutable - Your system's desired state should be stored in a way that ensures it remains unchanged (**immutable**) and maintains a complete history of changes (**versioned**). By preserving the history of changes you can track and understand the evolution of your system over time.
* Pulled Automatically - Agents or tools should automatically retry and apply the desired state from a specified source (Git usually). Simplifies the process of updating and maintaining your system, reducing the risk of human error.
* Continuously Reconciled - Agents or tools constantly **monitor** the **actual state** of the system and compare it against the desired state. This ensures that your system remains in the desired state at all times.

</details>

<details>
  <summary>Related Practices</summary>

* Configuration as Code (CaC)
  * Involves managing and provisioning infrastructure through machine-readable configuration files.
* Infrastructure as Code (IaC)
  * Key practice within DevOps that involves managing and provisioning computing infrastructure through code instead of manual processes.
  * Foundational to GitOps.
* DevOps and DevSecOps
  * DevOps is a set of practices that combines software development (Dev) and IT operations (Ops) to shorten the development lifecycle and provide continuous delivery with high software quality.
  * DevSecOps extends DevOps by integrating security practices into the DevOps process.
* CI/CD
  * CI - Continuous Integration, the automated process of integrating code changes from multiple contributors into a shared repository.
  * CD - Continuous Delivery, the practice of automating the software delivery progrss to ensure that code changes can be deployed to production at any time.

</details>

<details>
  <summary>GitOps Patterns</summary>

* Deployment and Release Patterns
  * Recreate - All or nothing.
  * Rolling - Gradually replace old with new.
  * Blue/Green - Two identical environments, one active and one idle. Switch traffic to the new version.
* Progressive Delivery Patterns
  * Continuous Delivery by gradually rolling out new features or updates to a subset of users!
  * Canary - Gradually roll out the new version to a small subset of users before full deployment.
  * Shadow - Run the new version alongside the old one without exposing it to users. Useful for testing and validation.
  * A/B Testing - Compare two versions of a system by directing a portion of users to each version. Helps in making data-driven decisions about which version to deploy.
* Pull vs. Event-driven
  * Pull based is required for GitOps, agents within the cluster coninuously monitor the Git repository for changes and apply these automatically.
  * Event driven, while not the primary model in GitOps, event-driven mechanisms can complement GitOps by triggering actions based on specific events.
* Architecture Patterns (in-cluster and external reconciler, state store management, etc.)
  * In-cluster Reconciler - A software agent that runs iwthin the cluster and is responsible for monitoring the state of the cluster.
  * External Reconciler - Similar to in-cluster reconcilers, but run outside the cluster. Often used for multi-cluster managment.
  * State Store Management - Structuring and organizing the state store to ensure immutability, versioning and complete version history.
  * Secrets Management - Managing secrets in a secure and compliant manner, often using tools like Vault, Sealed Secrets or KubeSecrets.
  
</details>

<details>
  <summary>Tooling</summary>

* Manifest Format and Packaging
  * Kustomize - Offers a template-free way to customize application configuration that simplifies the declaration of application manifests.
  * Helm - Provides packaging of Kubernetes applications into charts.
  * Declaritive YAML/JSON - A format for expressing the desired state of a system in a way that is both human-readable and machine-readable.
* State Store Systems (Git and alternatives)
  * Git - Distributed version control system, example of a state store used in GitOps.
  * OCI - Open Container Initiative, a set of standards for container images and runtimes. Stores and distribute container images.
* Reconciliation Engines (ArgoCD, Flux, and alternatives)
  * ArgoCD - A declarative, GitOps continuous delivery tool for Kubernetes.
  * Flux - A tool for keeping Kubernetes clusters in sync with sources of configuration.
  * Jenkins X - An open-source system that provides pipeline automation for Kubernetes.
* Interoperability with Notifications, Observability, and Continuous Integration Tools
  * DORA Metrics - DevOps Research and Assessment metrics that measure software delivery performance.
    * Deployment Frequency - How often an organization successfully releases to production.
    * Lead Time for Changes - Time it takes to go from code commit to code running in production.
    * Change Failure Rate - Percentage of changes that fail.
    * Time to Restore Service
    * MTTR - Mean Time to Recovery
  * Keptn - Integrates with Flux and ArgoCD to provide automated continuous delivery and operations for cloud-native applications.

</details>
