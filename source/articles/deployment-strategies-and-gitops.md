# Deployment strategies and GitOps

**Covers:** rolling, canary, blue-green, GitOps

## The same release, three levels of risk {#overview}
Putting a new version live is the moment of greatest risk in software delivery. If the new code has a bug, every user may see it at once. Deployment strategies are ways to limit that damage. Each one trades speed, cost and simplicity against how much harm a bad release can do. GitOps adds a different idea: where the truth about what should be running is kept. Let's take the three strategies first.

## rolling {#rolling}
A rolling deployment replaces the old instances of an application with new ones gradually, one or a few at a time, instead of all at once. Say you have 10 instances and a batch size of 2. Harness replaces 2, waits for them to become healthy, replaces the next 2, and continues until all are new.

It needs no extra capacity beyond a small margin, and the application stays up throughout. There are two drawbacks. Old and new versions run side by side for a while, so they must be compatible, for example in how they use the database. And to undo a rollout, you have to roll again in the other direction.

## canary {#canary}
A canary deployment sends the new version to a small slice of real traffic first, such as 5 percent of users, and checks that all is well before sending more. The name comes from the canary that miners once took underground to warn them of bad air.

If the canary's metrics look bad, such as more errors or slower responses, the release stops. Traffic goes back to the old version, so only a few users were affected. If the metrics look good, the share grows in steps, for example 5, 25 and then 100 percent. The cost is a more complex setup, a longer release, and the need for good measurements to decide.

## blue-green {#blue-green}
A blue-green deployment keeps two complete environments: the current one, say blue, and a new one, green. You deploy the new version to the idle green environment, test it there, and then switch all the traffic from blue to green at once. Blue remains available.

The main benefit is a fast and clean way back. If green has a problem, you switch the traffic back to blue within seconds. The costs are running two full environments at the same time, which is expensive, and the care needed with databases, because both versions share the same data.

## GitOps {#gitops}
GitOps is a deployment approach in which a Git repository is the single source of truth for what should be running. An automatic agent keeps making the real system match it. To change production, you change a file in Git and have the change reviewed and merged. You do not click a button or run a command on a server.

The benefits are a full history of every change, easy reviews, and a way back by reverting a commit. It also detects drift. If someone changes the system by hand, the agent notices that it no longer matches Git and corrects it. Harness supports it with a GitOps module built around the Argo CD project.

## A worked example {#example}
A service has 10 instances and 10,000 requests per minute. Let's compare how each strategy exposes users to a faulty release.

```python
traffic = 10_000
for percent in (5, 25, 100):
    print(percent, traffic * percent // 100)
# 5 500
# 25 2500
# 100 10000
```

The canary steps show how many requests per minute reach the new version: 500 at 5 percent, 2,500 at 25 percent, and all 10,000 at 100 percent.

| Strategy | Users exposed to a bad version before detection | Undo | Extra capacity needed |
|---|---|---|---|
| Rolling (2 of 10 at a time) | About 20 percent at the first batch, growing | Roll back through the batches | Small |
| Canary (5 percent first) | About 5 percent, 500 of the 10,000 requests per minute | Send the traffic back | Small |
| Blue-green | None during testing, then 100 percent at the switch | Switch back in seconds | A full second environment |

Now suppose a bug makes 30 percent of requests fail in the new version. With a canary at 5 percent, the failures are 30 percent of 500, which is 150 failed requests per minute. If monitoring notices within two minutes, that is about 300 failures in all. A release to everyone at once would fail 3,000 requests per minute (30 percent of 10,000), twenty times the damage per minute.

With GitOps, the rollout itself is a Git change: you edit the version number in the deployment file and merge it. To undo it, you revert the commit, and the agent restores the previous version.

## A common mistake {#mistake}
A common slip is to choose a canary without the measurements to judge it. If nobody looks at the error rate, a canary is only a slower release. Another slip is to forget that two versions run together during rolling and canary releases, so a change to the database must work for both.

## Key points {#key-points}
- Rolling replaces instances in batches. Canary tests a small share of traffic first. Blue-green switches between two complete environments.
- Each trades speed, cost and safety differently.
- Always decide with real metrics, and design changes so that two versions can coexist.
- GitOps makes Git the source of truth, and an agent keeps the real system matching it.
