# Delegates, selectors and connectors

**Covers:** Delegate, selector, connector, SaaS, Kubernetes, GitHub, API, IDs

## How a cloud platform reaches your private systems {#overview}
Harness is offered as a service on the internet, but your code, your clusters and your servers are usually inside your own private network, behind firewalls. The platform must be able to act on them, without you opening those networks to the world. The answer is a small program that you run inside your network, plus saved connection details for each outside tool.

## SaaS {#saas}
SaaS, or software as a service, means that the software runs on the provider's own servers and you use it over the internet, instead of installing and operating it yourself. Harness's main product is offered this way. The provider handles upgrades and capacity, and you pay for use. The trade-off is that the control center is outside your network, which is why a component is needed to reach what is inside.

## Kubernetes {#kubernetes}
Kubernetes is a system for automatically running, scaling and managing containerized applications across a group of machines. In Harness it plays two roles: it is a common place to deploy your applications, and it is a common place to run the Delegate itself. You do not need to know it deeply for this article, only that it hosts programs as units called pods.

## Delegate {#delegate}
The Delegate is a lightweight worker program that you run inside your own network, for example as a program inside your Kubernetes environment. It is Harness's hands in your infrastructure. The Harness service decides what must be done, and the Delegate does it: it clones code, runs build tasks, applies deployments and calls internal systems.

The important design feature is that the Delegate makes only outgoing connections to Harness. It asks "is there any work for me?" over an encrypted connection, so you do not open any inbound port or firewall hole. Credentials for your internal systems stay inside your network, because the Delegate uses them locally. You can run several Delegates for reliability and for different networks.

## selector {#selector}
A selector is a tag that you put on a Delegate, so that Harness can send a task to that particular Delegate. When you have several Delegates, for example one for production and one for a lab, you use selectors to direct work: a deployment to production uses the `prod` selector, so only the production Delegate takes the job. Selectors can be set at the pipeline, stage or step level.

## connector {#connector}
A connector is a saved, reusable object that stores the credentials and settings for reaching an outside system, such as GitHub, Docker Hub, a cloud account or a Kubernetes environment. You create it once, and any pipeline can refer to it by its name.

That avoids pasting passwords into pipelines, and it centralizes changes: when a token expires, you update the connector in one place, and every pipeline that uses it keeps working. A connector can be set to use a particular Delegate, which decides from where the connection is made.

## GitHub {#github}
GitHub is a popular website for storing Git repositories, where teams review code and automate work. It is one of the most common sources of code for Harness. A GitHub connector lets a pipeline clone a repository, and lets Harness receive notifications of pushes and pull requests.

## API {#api}
An API, or application programming interface, is a defined way for one program to ask another for something. Connectors are built on APIs: the GitHub connector uses GitHub's API, with a token, to read code and report status. Harness also has its own API, so that you can create and run pipelines from scripts.

## IDs {#ids}
IDs, or identifiers, are the unique names or codes that tell one item apart from another. Every Harness object, such as a pipeline, a connector or a Delegate, has an identifier. It is fixed when you create it, can contain only letters, digits and underscores and is what other files use to refer to the object. The display name can be changed later, but the identifier cannot, so choose it with care.

## A worked example {#example}
A company wants a pipeline to build code from GitHub and deploy to a private environment.

| Step | Action |
|---|---|
| 1 | Install a Delegate inside the private environment. It connects out to Harness and appears as healthy |
| 2 | Create a GitHub connector with identifier `github_main`, using a token stored as a secret |
| 3 | Create a Kubernetes connector `k8s_prod` that uses the Delegate's internal access |
| 4 | Tag the production Delegate with the selector `prod` |
| 5 | In the pipeline, refer to `github_main` and `k8s_prod`, and use the selector `prod` |

What happens at run time: Harness (SaaS) decides that the deploy step must run. It queues the task for a Delegate matching `prod`. The Delegate picks it up through its outgoing connection and applies the deployment inside the private environment. The firewall has no inbound hole.

Count the benefit: with 15 pipelines using GitHub, a rotated token needs 1 change to the connector. Without connectors, it would be 15 edits. And if two Delegates exist (prod and lab), the selector guarantees that a lab job can never touch production.

## A common mistake {#mistake}
Running a single Delegate in production, which is a single point of failure. Run at least two. Another mistake is choosing a vague identifier such as `test1`, which cannot be changed later, so that it confuses everyone for years.

## Key points {#key-points}
- A Delegate runs in your network and connects out to Harness, so no inbound firewall holes are needed.
- Selectors are tags that direct a task to the right Delegate.
- Connectors store credentials once for tools such as GitHub, and pipelines refer to them by identifier.
- Identifiers are permanent, so choose them with care.
