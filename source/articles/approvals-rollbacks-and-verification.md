# Approvals, rollbacks and continuous verification

**Covers:** approval, rollback, health check, Continuous Verification

## Safety nets for a deployment {#overview}
Automation is fast, and fast mistakes are expensive. Three mechanisms add safety without taking away the speed: a person who can say "go" at the right moment, a machine that can tell when something is wrong and a way to go back to the last good state. This article covers each, and how they work together.

## approval {#approval}
An approval is a pipeline step that pauses the run and waits for a specific person or group to say "go" before it continues. The approvers can be named users, user groups or people in an external system such as Jira or ServiceNow. The step shows them what is about to happen, and they approve or reject it, with a comment.

Typical uses are before a deployment to production, before a costly operation or when rules require a second pair of eyes. Good practice is to set a timeout, so a forgotten approval does not leave a pipeline waiting forever, to require approval from someone other than the author of the change and to make the approval request informative, with the version and the list of changes.

## health check {#health-check}
A health check is an automatic probe that repeatedly asks whether something is still working correctly, for example by requesting a `/health` address and expecting a 200 response. Kubernetes uses them to decide whether to send traffic to an instance or to restart it. Harness uses them after a deployment to decide whether the new version is fit to continue.

A good health check tests something real, such as whether the application can reach its database, and not only whether the process is alive. A check that always says "healthy" is worse than none.

## Continuous Verification {#continuous-verification}
Continuous Verification is a Harness CD feature that watches live metrics and logs during a deployment, compares them with the behavior before the change, and flags or stops a bad release automatically. It reads from your monitoring tools, such as Prometheus, Datadog or New Relic, and uses statistical analysis to judge whether error rates, response times or log patterns are significantly worse than the baseline.

It replaces the human habit of staring at dashboards after a release. The step can fail the deployment, and trigger a rollback, if the numbers go wrong. It works best with strategies such as canary, where the new version can be compared with the old one at the same moment.

## rollback {#rollback}
A rollback returns a deployment to its last known-good version, either triggered by a person or automatically when something fails. Harness keeps the information about the previous successful deployment, so a rollback can redeploy it without anyone searching for the old version.

Plan for it in advance. Problems appear when the change cannot be undone: a database change that deletes a column cannot be reversed by redeploying old code. Make such changes in steps, adding first and removing later, so that both the old and the new code work with the data.

## A worked example {#example}
A production deploy stage with all four parts:

```yaml
steps:
  - step:
      type: HarnessApproval
      name: Approve production release
      spec:
        approvers:
          userGroups: [release_managers]
        approverInputs: []
      timeout: 1h
  - step:
      type: Deploy
      name: Canary 10 percent
  - step:
      type: Verify
      name: Verify canary
      spec:
        type: Canary
        duration: 10m
  - step:
      type: Deploy
      name: Complete rollout
failureStrategies:
  - onFailure:
      errors: [AllErrors]
      action:
        type: StageRollback
```

Follow it:

1. The pipeline pauses at the approval step. A release manager reads the version and the change list and approves within 20 minutes. If nobody answered within one hour, the step would time out.
2. The canary step deploys the new version to 10 percent of the instances. The health check keeps the instance out of service until it passes, so a broken start does not receive traffic.
3. Continuous Verification watches the metrics for 10 minutes and compares the canary with the old version. Suppose the error rate rises from 0.5 to 4 percent. The step fails.
4. The failure strategy triggers a stage rollback, and the previous version is redeployed. Users saw the problem for about 10 minutes, on only 10 percent of the traffic.

Count the exposure: if the service handles 6,000 requests per minute, 10 percent is 600, and with a 4 percent error rate that is 24 failed requests per minute, or about 240 in 10 minutes. A full release with the same bug, found by a person after 20 minutes, would fail 4 percent of 6,000, which is 240 per minute, or 4,800 in 20 minutes. That is 20 times more failures.

## A common mistake {#mistake}
An approval that is always clicked without reading, such as a "yes" on a long list of changes. It adds delay without adding safety. Show what matters. Another mistake is a rollback plan that has never been tried. Practice it in staging before you need it.

## Key points {#key-points}
- An approval pauses a pipeline for a person to decide. Set a timeout and show clear information.
- A health check should test something real, not just that the process runs.
- Continuous Verification compares live metrics with a baseline and can stop a release.
- Rollback returns to the last good version, so design changes that can be undone.
