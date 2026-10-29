# Governance, policy as code and access control

**Covers:** governance, OPA, Policy as Code, RBAC

## Freedom with guard rails {#overview}
When many teams can create and run pipelines, someone has to make sure that company rules are followed: production deploys need an approval, only approved images are used, no one deploys on Friday evening. Relying on every team to remember is not enough. Harness offers two controls: automatic checks of pipelines against written rules, and a model that decides who is allowed to do what.

## governance {#governance}
Governance, in Harness, means automatically checking pipelines against a set of written rules, called policies, before they are allowed to run, so that mistakes and risky shortcuts are stopped early. A policy can say that every production stage must contain an approval step, that images must come from the company registry, or that a pipeline must not use a certain unsafe setting.

The check happens at defined moments, such as when a pipeline is saved or before it runs. A failing check can block the action or just warn, depending on how strict you set it. This turns rules from a document that people may not read into something the platform enforces.

## OPA {#opa}
OPA, the Open Policy Agent, is an open-source policy engine, and Harness governance is built on it. OPA evaluates data, here the definition of a pipeline, against rules written in a language called Rego. It answers one question: allowed or not allowed, with a message that explains why.

Because it is a general standard, used also for Kubernetes and other systems, skills and policies are reusable across them.

## Policy as Code {#policy-as-code}
Policy as Code means writing your organization's rules as real, version-controlled code, instead of as documents or habits. Policies are stored in files, reviewed like any change, tested and rolled out in steps, and every change has a history and an author. That makes rules precise and checkable, and removes disagreements about what the rule says.

## RBAC {#rbac}
RBAC, or role-based access control, is a permissions system in which what a user can do is decided by the roles they hold, not by permissions given one by one. A role is a set of permissions, such as "can view pipelines" or "can run deployments to staging". A user, or a group, is assigned roles over a scope, which in Harness can be the whole account, an organization or a single project.

It keeps permissions manageable: when a person changes teams, you change their group, and their rights follow. The guiding rule is least privilege: give each person only what they need.

## A worked example {#example}
A company wants to enforce that every production deployment has an approval step.

A policy written in Rego:

```text
package pipeline

deny[msg] {
    stage := input.pipeline.stages[_].stage
    stage.type == "Deployment"
    stage.spec.environment.type == "Production"
    not has_approval(stage)
    msg := sprintf("Stage '%s' deploys to production without an approval step", [stage.name])
}

has_approval(stage) {
    stage.spec.execution.steps[_].step.type == "HarnessApproval"
}
```

How it is used:

1. The policy is saved in Harness and attached to a policy set, which applies it when a pipeline is saved or run.
2. A developer builds a pipeline that deploys to production with no approval step and tries to run it.
3. Harness sends the pipeline to OPA, which finds a match for the `deny` rule and returns the message: "Stage 'Deploy prod' deploys to production without an approval step".
4. The run is blocked until the developer adds the approval.

Now access. The roles are assigned like this:

| Person | Group | Role | Scope |
|---|---|---|---|
| Ana (developer) | `shop-devs` | Pipeline Executor | Project Shop |
| Ben (release manager) | `release-managers` | Pipeline Admin and Approver | Project Shop |
| Cara (auditor) | `auditors` | Viewer | Whole account |

Ana can run pipelines in her project, but she cannot change who may approve. Cara can look at everything and change nothing.

Count the benefit of Policy as Code. If there are 60 pipelines, a manual review for the approval rule would take about 5 minutes for each, which is 300 minutes or 5 hours, and it would have to be repeated whenever a pipeline changed. The policy checks all of them in seconds, every time.

## A common mistake {#mistake}
Writing policies in blocking mode right away, which stops teams in the middle of their work. Start in warning mode, fix the existing violations and then turn on blocking. Another mistake is giving everyone the administrator role because it is easy. Use groups and the least privilege.

## Key points {#key-points}
- Governance checks pipelines against rules automatically before they run.
- OPA is the open-source engine, and policies are written in Rego.
- Policy as Code keeps rules in version control, so they are precise, reviewed and testable.
- RBAC assigns roles at a scope. Give the least privilege that is needed.
