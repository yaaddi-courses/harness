# Triggers: starting pipelines automatically

**Covers:** trigger, cron trigger, PR, webhook, HTTP

## A pipeline nobody has to remember to run {#overview}
A pipeline that has to be started by a person will sometimes be forgotten. Triggers remove that step: they watch for an event and start the pipeline when it happens. The event may be a change in the code, a clock reaching a certain time or a notification from another system. This article covers the kinds and the technology behind the most common one.

## trigger {#trigger}
A trigger is a rule that starts a pipeline automatically when something happens, such as a push to Git, a pull request, a new artifact or a schedule. You attach triggers to a pipeline, and each one has a type, conditions and the input values to use. A pipeline can have several triggers, for instance one for pushes to the main branch and one for a nightly run.

Conditions make triggers selective: only for a certain branch, only when files in a certain folder changed, only for a certain tag. Without them, a busy repository would start runs that nobody needs.

## PR {#pr}
A PR, or pull request, is a proposal to merge the changes from one branch into another, opened for review. A trigger on pull requests runs the build and tests on the proposed change before anyone merges it, and reports the result back on the PR as a check. That means broken code is found before it reaches the main branch. Typical events are a PR being opened, updated with new commits or merged.

## cron trigger {#cron-trigger}
A cron trigger starts a pipeline on a fixed schedule, such as every night at 2 a.m., using the same time-pattern syntax as the classic Unix cron tool. A pattern has five fields: minute, hour, day of the month, month and day of the week. For example `0 2 * * *` means minute 0 of hour 2, every day. An asterisk means "any value".

Scheduled runs suit work that is not tied to a code change: a nightly full test suite, a weekly security scan, a regular clean-up. Be careful with time zones, since the pattern is read in a stated zone, and with daylight saving changes.

## HTTP {#http}
HTTP, or HyperText Transfer Protocol, is the set of rules that web browsers and servers use to exchange pages and data. It is the language of most automatic notifications between systems: one program sends a request to a web address, with some data, and the other answers with a status code, such as 200 for success. Understanding it helps with webhooks.

## webhook {#webhook}
A webhook is an automatic HTTP call that one system makes to another when something happens. For example, GitHub notifies Harness that a push occurred by sending a small message to a web address that Harness provides. Instead of Harness repeatedly asking "has anything changed?", which is called polling and wastes effort, the source system tells Harness at once.

A webhook trigger has a unique URL, and you register it in the code host. Because anyone who knows the address could send a request, protect it: use a secret that the sender signs the message with, and check the signature.

## A worked example {#example}
Three triggers for one pipeline:

| Trigger | Type | Condition | Effect |
|---|---|---|---|
| `on_pr` | Webhook (GitHub pull request) | Target branch `main`, event opened or updated | Build and test each proposed change |
| `on_merge` | Webhook (GitHub push) | Branch `main` | Build, test and deploy to staging |
| `nightly` | Cron | `0 2 * * *` | Run the full test suite |

A cron pattern is read like this:

```text
0 2 * * *
| | | | |
| | | | +-- day of the week (any)
| | | +---- month (any)
| | +------ day of the month (any)
| +-------- hour: 2
+---------- minute: 0
```

Trace one event. A developer opens a PR. GitHub sends an HTTP request to the webhook address. Harness checks the signature, matches the `on_pr` trigger and starts a run for that commit. Within 7 minutes, a green check shows on the PR, or a red one with a link to the logs.

Compare with polling every minute: with 20 repositories, that is 20 requests a minute, 1,200 an hour and 28,800 a day, nearly all of them answered "nothing new". With webhooks, there is one request for each real event, perhaps 100 a day, and the reaction is immediate.

The nightly cron trigger needs no event at all. At 2 a.m., it starts the 90-minute full suite, so the results are ready when the team arrives.

## A common mistake {#mistake}
Triggering on every push to every branch, which floods the system with runs that nobody needs. Add branch and path conditions. Another mistake is exposing a webhook address with no signature check, so that anyone could start your pipeline.

## Key points {#key-points}
- A trigger starts a pipeline automatically when an event or a schedule fires.
- A PR trigger tests a proposed change before it is merged.
- A cron trigger uses five fields to run on a schedule, and the time zone matters.
- A webhook is an HTTP notification from one system to another. It beats polling, and should be signed.
