# Security scans, chaos engineering and cloud cost

**Covers:** STO, UI, Chaos Engineering, pod, CCM, AWS

## Beyond build and deploy {#overview}
Harness grew from a deployment tool into a platform with modules for other parts of the software life: finding security flaws, proving that a system survives failures and keeping cloud bills under control. Each runs on the same pipeline model and the same permissions as the rest. This article describes three of them, with the words that go along with them.

## STO {#sto}
STO, or Security Testing Orchestration, is the Harness module that runs security scans as steps inside pipelines and gathers their results in one place. Instead of each team installing and running its own scanners, the pipeline calls them: tools that look for flaws in your code (static analysis), in your running application (dynamic testing), in your dependencies and in your container images.

The results from different scanners are combined and de-duplicated, so that you see one list of issues and not ten separate reports. You can set rules: a pipeline fails if a critical flaw is found, or if a new high-severity flaw appears compared with the last run.

## UI {#ui}
UI means user interface: the screens and controls through which people use software. In this context, it matters in two ways. Harness has a visual editor that shows the pipeline as boxes you can drag, alongside the YAML. And security and testing tools often scan a web application's UI, by driving it like a user to find weaknesses, which is how dynamic testing works.

## pod {#pod}
A pod is the smallest deployable unit in Kubernetes: one running instance of your containerized application, or a small group of tightly coupled containers that share a network address. Pods are temporary. They are created, destroyed and replaced often, which is exactly what makes them a good target for chaos tests: a healthy system must survive losing one.

## Chaos Engineering {#chaos-engineering}
Chaos Engineering is the practice of deliberately injecting controlled failures, such as killing a server, adding network delay or filling a disk, to learn whether a system really survives them. The idea is that you would rather find the weakness on a Tuesday afternoon, with the team watching, than at 3 a.m. during a real outage.

The method has four steps:
1. Define what normal looks like, for example "99 percent of checkouts succeed within 2 seconds".
2. Form a hypothesis: "if we kill one payment pod, the success rate stays above 99 percent".
3. Run the experiment, with a limit on its size and a way to stop it.
4. Compare and learn. If the hypothesis was wrong, fix the weakness and test again.

The Harness module provides ready-made experiments, such as pod deletion and network latency, and can run them as a step in a pipeline, with a check on the results. Start small, in a test environment, before you go near production.

## AWS {#aws}
AWS, or Amazon Web Services, is Amazon's cloud platform for renting servers, storage and many other services. Much of the world's software runs on it, as it does on Microsoft Azure and Google Cloud. It matters for these modules because chaos experiments often target cloud resources, and cost management reads the AWS bill.

## CCM {#ccm}
CCM, or Cloud Cost Management, is the Harness module for tracking and cutting cloud spending. It reads the billing data from your cloud accounts and Kubernetes clusters, splits the cost by team, project or application and points out waste: machines that sit idle, resources nobody uses, oversized servers.

It can also shut down non-production resources outside working hours, and give a team a budget with an alert. Assigning costs to their owners is half of the work.

## A worked example {#example}
A team uses all three modules around one service.

| Module | What the team does | Result |
|---|---|---|
| STO | Adds a scan step to the pipeline; sets "fail on any critical flaw" | A build is blocked because a library has a critical flaw; the team updates it |
| Chaos Engineering | Experiment: delete one of 3 `payment` pods | Success rate falls to 91 percent for 40 seconds, below the 99 percent goal: a weakness |
| CCM | Reviews the monthly cost, finds a staging cluster that is idle at night | A schedule turns it off from 8 p.m. to 7 a.m. |

Work through the chaos result. With 3 pods and one deleted, the remaining 2 must carry the load. Why did the success rate fall? The team finds that the readiness check was too slow to remove the dead pod from the load balancer, so for 40 seconds a third of the requests went to a pod that no longer existed: 1/3 is 33 percent of requests, but retries hid most of them, leaving a 9 percent failure rate (100 minus 91). They shorten the check interval, repeat the experiment and see 99.5 percent.

Now the cost. The idle staging cluster costs 12 dollars an hour. Turning it off for 11 hours a night, 7 days a week, saves 11 x 12 x 7 = 924 dollars a week, about 4,000 dollars a month.

All three results came from the same pipeline and permission model that the team already knew.

## A common mistake {#mistake}
Running chaos experiments in production first, with no limit and no way to stop them. Start in a test environment with a small experiment. Another mistake is treating a clean scan as proof of security. Scanners find known problems only, and they miss design flaws.

## Key points {#key-points}
- STO runs and combines security scans inside pipelines and can fail a build on serious findings.
- Chaos Engineering injects controlled failures to test a hypothesis about resilience. Start small.
- A pod is a short-lived Kubernetes unit, so a healthy system survives losing one.
- CCM tracks cloud cost by owner and finds waste, such as idle resources.
