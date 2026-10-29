# Approvals, rollbacks and continuous verification

**Covers:** approval, rollback, health check, Continuous Verification

## Safety nets for a deployment {#overview}
Automation is fast, and fast mistakes are expensive. Think about what you would want on release day: a person who can say "go" at the right moment, a machine that can tell when something is wrong, and a way back to the last good state. Those are three mechanisms, and each one adds safety without taking away the speed. Let's look at them one at a time, and then at how they work together.

## approval {#approval}
An approval is a pipeline step that pauses the run until a specific person or group says "go". The approvers can be named users, user groups or people in an external system such as Jira or ServiceNow. The step shows them what is about to happen, and they approve or reject it, with a comment.

You will typically use one before a deployment to production, before a costly operation, or when the rules require a second pair of eyes. Three habits make approvals work. Set a timeout, so a forgotten approval does not leave a pipeline waiting forever. Require approval from someone other than the author of the change. And make the request informative, with the version and the list of changes.

## health check {#health-check}
A health check is an automatic probe that asks, again and again, whether something still works correctly. A common form requests a `/health` address and expects a 200 response. Kubernetes uses health checks to decide whether to send traffic to an instance or restart it. Harness uses them after a deployment to decide whether the new version is fit to continue.

A good health check tests something real, such as whether the application can reach its database, and not only whether the process is alive. A check that always says "healthy" is worse than none, because it gives you confidence you have not earned.

## Continuous Verification {#continuous-verification}
Continuous Verification is a Harness CD feature that watches live metrics and logs during a deployment and compares them with how things looked before the change. If the release looks bad, it flags it or stops it automatically. It reads from your monitoring tools, such as Prometheus, Datadog or New Relic, and uses statistical analysis to judge whether error rates, response times or log patterns are significantly worse than the baseline.

It replaces a human habit: staring at dashboards after a release. The step can fail the deployment, and trigger a rollback, if the numbers go wrong. It works best with strategies such as canary, where the new version can be compared with the old one at the same moment.

## rollback {#rollback}
A rollback returns a deployment to its last known-good version. A person can trigger it, or it can run automatically when something fails. Harness keeps the details of the previous successful deployment, so a rollback can redeploy it and nobody has to search for the old version.

Plan for it in advance, because some changes cannot be undone. A database change that deletes a column cannot be reversed by redeploying old code. Make such changes in steps, adding first and removing later, so that both the old and the new code work with the data.

## A worked example {#example}
Here is a production deploy stage with all four parts:

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

Let's follow a release through it.

1. The pipeline pauses at the approval step. A release manager reads the version and the change list and approves within 20 minutes. If nobody answered within one hour, the step would time out.
2. The canary step deploys the new version to 10 percent of the instances. The health check keeps an instance out of service until it passes, so a broken start does not receive traffic.
3. Continuous Verification watches the metrics for 10 minutes and compares the canary with the old version. Suppose the error rate rises from 0.5 to 4 percent. The step fails.
4. The failure strategy triggers a stage rollback, and the previous version is redeployed. Users saw the problem for about 10 minutes, on only 10 percent of the traffic.

Now count the exposure. If the service handles 6,000 requests per minute, 10 percent is 600, and with a 4 percent error rate that is 24 failed requests per minute, or about 240 in 10 minutes. Suppose instead the same bug went to everyone and a person noticed it after 20 minutes. Then 4 percent of 6,000 is 240 failed requests per minute, or 4,800 in 20 minutes. That is 20 times more failures.

## A common mistake {#mistake}
A common slip is an approval that is always clicked without reading, such as a "yes" on a long list of changes. It adds delay and no safety. Show what matters. Another slip is a rollback plan that nobody has tried. Practice it in staging before you need it.

## Key points {#key-points}
- An approval pauses a pipeline for a person to decide. Set a timeout and show clear information.
- A health check should test something real, and not only that the process runs.
- Continuous Verification compares live metrics with a baseline and can stop a release.
- Rollback returns to the last good version, so design changes that can be undone.
