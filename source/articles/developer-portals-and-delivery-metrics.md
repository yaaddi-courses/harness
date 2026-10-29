# Developer portals and delivery metrics

**Covers:** IDP, golden path, golden-path, SEI, DORA, DORA metrics, cycle time

## Making the right way the easy way {#overview}
As a company grows, so does the number of services, tools and rules. A new developer can spend days only on finding out how to create a service that is allowed to run in production. Two more Harness modules deal with both sides of this problem. One is a portal that makes the recommended way easy. The other is a set of measurements that show whether delivery is really improving. Let's start with the portal.

## IDP {#idp}
An IDP, or internal developer portal, is a single place where developers find services, documentation and templates, and can start new things on their own. Typical contents are a catalog of all services with their owners, links to their docs and dashboards, and forms that create things: "new service", "new database", "new pipeline". The Harness module is built on the open-source Backstage project.

The goal is self-service. A developer who wants a new service fills in a short form, and the portal creates the repository, the pipeline and the monitoring, all set up the approved way, in minutes and with no ticket to another team.

## golden path {#golden-path}
A golden path is a ready-made, pre-approved template, offered through the portal, that shows the recommended and supported way to build and ship something. It is "golden" because it is the easiest road. If you follow it, you get security settings, a pipeline, monitoring and documentation from the start.

It works by making the good choice the convenient one. It is also optional, so teams with special needs can leave it, knowing that they take on the extra work. A golden path must be kept up to date, or it becomes the bad choice.

## golden-path {#golden-path-hyphen}
You will also see the hyphenated spelling, and it means the same thing: a recommended, pre-approved way of building and shipping software that platform teams make easy to follow. Documents use both spellings.

## DORA {#dora}
DORA stands for DevOps Research and Assessment. It is the research group, begun in the 2010s and now part of Google Cloud, that studied thousands of teams to find out what separates high performers from the rest. Its yearly State of DevOps report is widely cited.

## DORA metrics {#dora-metrics}
DORA metrics are four standard measures of software delivery performance:
1. **Deployment frequency:** how often you release to production.
2. **Lead time for changes:** how long it takes from a code change being committed to its running in production.
3. **Change failure rate:** the share of releases that cause a problem needing a fix or a rollback.
4. **Time to restore service:** how long it takes to recover after a failure.

The first two measure speed, and the last two measure stability. The research finding that surprised many people is that the best teams are better at both at once, because speed and stability support each other.

## cycle time {#cycle-time}
Cycle time is, roughly, how long a piece of work takes to go from someone starting on it to its being delivered. It counts waiting as well as working: time in review queues, in test and in a release train. It is often drawn as a breakdown, so you can see where the time goes. If the cycle time is long and the working time is short, the delays are in the process and not in the coding.

## SEI {#sei}
SEI, or Software Engineering Insights, is the Harness module that measures how engineering teams deliver. It pulls data from the code host, the ticket tracker and the pipelines, and it shows DORA metrics, cycle time and trends for each team. Use it to find bottlenecks and to see whether a change in process really helps. Do not use it to rank individuals, which tends to ruin the data.

## A worked example {#example}
Here are a team's last four changes, as commit time, deploy time and whether the deployment failed:

```python
from datetime import datetime

changes = [
    ("2026-10-01 10:00", "2026-10-01 15:00", False),
    ("2026-10-03 09:00", "2026-10-04 09:00", True),
    ("2026-10-08 11:00", "2026-10-08 12:00", False),
    ("2026-10-10 14:00", "2026-10-10 18:00", False),
]


def hours(start, end):
    fmt = "%Y-%m-%d %H:%M"
    delta = datetime.strptime(end, fmt) - datetime.strptime(start, fmt)
    return delta.total_seconds() / 3600


lead_times = [hours(c, d) for c, d, _ in changes]
print(sum(lead_times) / len(lead_times))
# 8.5
print(sum(failed for _, _, failed in changes) / len(changes))
# 0.25
```

Let's read the results.

1. The lead times are 5, 24, 1 and 4 hours, and their mean is 34 / 4 = 8.5 hours.
2. One of four changes failed, so the change failure rate is 1 / 4 = 0.25, or 25 percent.
3. Four deployments in the ten days from 1 to 10 October is 0.4 per day, or about one every 2.5 days.

Compared with the usual groupings in the DORA reports, a 25 percent failure rate is high, and a lead time of 8.5 hours is good but has room to improve. The team opens the cycle time breakdown in SEI for the 24-hour change and finds 2 hours of coding, 20 hours of waiting for review and 2 hours in test. The delay is in the review queue. So they set a rule to review within 4 hours, and they add a golden-path pipeline with automatic checks, which lowers the failure rate.

One more point: the average does not tell the whole story, so look at the slowest changes and not only at the mean.

## A common mistake {#mistake}
A common slip is to use DORA metrics to compare or rank people. Once a number becomes a target, people change their behavior to improve the number and not the work. Use the metrics to help teams see their own bottlenecks. Another slip is building a golden path and not maintaining it, so teams avoid it.

## Key points {#key-points}
- An IDP is a self-service portal with a catalog, docs and templates.
- A golden path is the easiest approved way to build and ship, and it must be kept current.
- DORA metrics are deployment frequency, lead time, change failure rate and time to restore.
- Cycle time shows where work waits. Use SEI to see the numbers, and use them to improve the process, not to rank people.
