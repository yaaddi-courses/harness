# Services, environments and infrastructure definitions

**Covers:** Service, Environment, Infrastructure Definition, cluster, VM

## Separating what, where and how {#overview}
In continuous delivery, three questions get mixed up easily. What are we deploying? Where does it go? And which exact machines are involved? Harness CD answers each question with its own object. Keeping them apart is what lets one pipeline deploy the same application to dev, staging and production, without being copied three times. Let's take the objects one by one.

## Service {#service}
In Harness CD, a Service defines what you are deploying: the application together with its artifact, such as a container image version, and its manifests or settings. It does not say where. A service called `shop-api` describes the application once, and the definition is the same wherever it goes.

The service can have several artifact versions, and you choose the version at run time. That makes "deploy version 2.4.1 of shop-api" a simple input to the pipeline.

## Environment {#environment}
An Environment is a named group of deploy targets, such as `staging` or `production`. It stands for a place and its rules, and it separates the different places the service may go. Environments carry their own settings, such as values and credentials, so the same service can use a test database in staging and the real one in production.

You class environments as production or non-production, which lets you apply stricter rules, such as a sign-off, to production ones.

## Infrastructure Definition {#infrastructure-definition}
An Infrastructure Definition is the exact deploy target within an environment: for example, which Kubernetes cluster and which namespace, or which group of virtual machines. An environment may have several. The `production` environment might contain one definition for a cluster in Europe and another for a cluster in the United States.

The pipeline picks the environment and the infrastructure definition, and Harness then knows exactly where to send the deployment.

## cluster {#cluster}
A cluster is a group of machines that work together as one system. In Kubernetes, it is the full set of servers that run your containers, managed as one pool. An infrastructure definition for Kubernetes usually names a cluster, through a connector, and a namespace inside it, which is a way of dividing a cluster into separate areas.

## VM {#vm}
A VM, or virtual machine, is a computer made of software that runs inside a real one. Deployments do not always use containers. Some teams deploy to a group of virtual machines, for example over SSH, and Harness supports this with a different kind of infrastructure definition, one that lists the hosts instead of a cluster.

## A worked example {#example}
A team deploys `shop-api` to staging and to production.

| Object | Value |
|---|---|
| Service | `shop-api`, artifact: image `shop-api` from the registry, version chosen at run time |
| Environment 1 | `staging` (non-production) |
| Infrastructure Definition 1 | Kubernetes cluster `eu-test`, namespace `shop` |
| Environment 2 | `production` (production) |
| Infrastructure Definitions 2 | Cluster `eu-prod`, namespace `shop`, and cluster `us-prod`, namespace `shop` |

One pipeline deploys the service, taking the environment as an input. Let's run it twice.

1. Select version `2.4.1` and environment `staging`. Harness deploys to the single staging cluster.
2. After testing, select version `2.4.1` and environment `production`. A sign-off step appears first, because it is a production environment. After the sign-off, Harness deploys to both production clusters.

Now count the objects. The service is defined once. There are 2 environments and 3 infrastructure definitions. Without this separation, you would need a pipeline for each target, 3 copies in all, and each would need the same update whenever the steps changed. Here, there is one pipeline and a few small definitions.

If production later gets a third region, you add one infrastructure definition and change no pipeline.

## A common mistake {#mistake}
A common slip is to put production and staging targets in the same environment, so the pipeline cannot tell them apart, and the extra sign-off rules apply to neither or to both. Keep the environments separate. Another slip is hard-coding the artifact version in the service. Let it be an input.

## Key points {#key-points}
- A Service defines what is deployed. An Environment is a named place. An Infrastructure Definition is the exact target in that place.
- A Kubernetes target is a cluster plus a namespace. Other targets are groups of VMs.
- One pipeline can deploy the same service to every environment.
- Production environments get stricter rules, such as sign-offs.
