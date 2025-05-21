# Certified Argo Project Associate

_This exam is an online, proctored, multiple-choice exam._

## Resources

* <https://argo-workflows.readthedocs.io/en/stable/>

## Topics

<details>
  <summary>Argo Workflows (36%)</summary>

* Understand Argo Workflow Fundamentals
* Generating and Consuming Artifacts
* Understand Argo Workflow Templates
* Understand the Argo Workflow Spec
* Work with DAG (Directed-Acyclic Graphs)
* Run Data Processing Jobs with Argo Workflows

Argo Workflows is an open source container-native workflow engine for orchestrating parallel jobs on Kubernetes. It's implemented as a Kubernetes CRD.

Workflows are implemented as a Kubernetes CRD.

* Lightweight and easy to use.
* Designed from the ground up for containers without the overhead and limitations of legacy VM and server-based environments.

Components of Argo Workflows include:

* controller
* mainContainer
* executor
* server

## Core Concepts

The `Workflow`is the most important resourcein Argo and serves two functions:

1. Itdefines the workflow to be executed.
2. It stores the state of the workflow.

Should be treated as a "live" object. It's not just a static definition but it's also an "instance" of said definition.

In the `WorkflowSpec` the core structures are:

* `entrypoint` defines what the main function will be - that is, the first template to be executed.
* `templates` can be loosely thought of as functions, they define instructions to be executed.

![alt text](image.png)

The internals of a step Pod are:

* `main` container runs the Image that the user has indicated, where the `argoexec` utility is volume mounted and serves as the main command which calls **the configured Command as a sub-process**.
* `init` container is an `InitContainer` fetching artifacts and parameters and making them available to the `main` container.
* `wait` container performs tasks that are needed for clean up, including saving of parameters and artifacts.

There are six types of templates, divided into two categories:

* Work to be done:
  * **Container** - The most common template type. The spec is the same as the one of a container spec in Kubernetes.
  * **Script** - Convenience wrapper around a `container`. The spec is the same as for a container but adds the `source:` field for in-place scripts. The script will be saved in a file and executed for you. The result of the script will be automatically exported into an Argo variable:

    ```
    {{tasks.<NAME>.outputs.result}}
    {{steps.<NAME>.outputs.result}}
    ```

  * **Resource** - Performs operations on cluster resources directly. It can be used to get, create, apply, delete replace or patch resouces on your cluster.
  * **Suspend** - Suspend execution, either for aduration or until it is resumed manually.

* Invocators:
  * **Steps** - A steps template you define your tasks in a series of steps. The structure of the template is a "list of lists". you can use synchronization to run the inner ones one by one. Control execution can be done with e.g. `when:`.
    * Outer lists will run sequentially
    * Inner lists will run in parallel
  * **DAG** - Define tasks as a graph of dependencies. In a DAG, you list all your tasks and set which other tasks must complete before a particular task can begin. Tasks without any dependencies will run in immedately.



</details>

<details>
  <summary>Argo CD (34%)</summary>

* Understand Argo CD Fundamentals
* Synchronize Applications Using Argo CD
* Use Argo CD Application
* Configure Argo CD with Helm and Kustomize
* Identify Common Reconciliation Patterns

</details>

<details>
  <summary>Argo Rollouts (18%)</summary>

* Understand Argo Rollouts Fundamentals
* Use Common Progressive Rollout Strategies
* Describe Analysis Template and AnalysisRun

</details>

<details>
  <summary>Argo Events (12%)</summary>

* Understand Argo Events Fundamentals
* Understand Argo Event Components and Architecture

</details>
