# Governance, policy as code and access control

**Covers:** governance, OPA, Policy as Code, RBAC

## Freedom with guard rails {#overview}
When many teams can create and run pipelines, someone has to make sure the company's rules are followed: production deploys need an approval, only approved images are used, and nobody deploys on a Friday evening. Relying on every team to remember is not enough. Harness gives you two controls for this. One checks pipelines automatically against written rules. The other decides who is allowed to do what.

## governance {#governance}
In Harness, governance means that pipelines are checked automatically against a set of written rules, called policies, before they may run. Mistakes and risky shortcuts are stopped early. A policy can say that every production stage must contain an approval step, that images must come from the company registry, or that a pipeline must not use a certain unsafe setting.

The check happens at defined moments, such as when a pipeline is saved or before it runs. Depending on how strict you set it, a failing check can block the action or only warn. This turns rules from a document that people may not read into something the platform enforces.

## OPA {#opa}
OPA, the Open Policy Agent, is an open-source policy engine, and Harness governance is built on it. OPA evaluates data, here the definition of a pipeline, against rules written in a language called Rego. It answers one question: allowed or not allowed, with a message that explains why.

OPA is a general standard that is also used for Kubernetes and other systems, so your skills and your policies carry over between them.

## Policy as Code {#policy-as-code}
Policy as Code means writing your organization's rules as real, version-controlled code, instead of documents or habits. The policies are stored in files, reviewed like any other change, tested and rolled out in steps, and every change has a history and an author. That makes the rules precise and checkable, and it ends arguments about what a rule says.

## RBAC {#rbac}
RBAC, or role-based access control, is a permissions system in which the roles a user holds decide what that user can do, instead of permissions given one by one. A role is a set of permissions, such as "can view pipelines" or "can run deployments to staging". You assign a user, or a group, roles over a scope. In Harness the scope can be the whole account, an organization or a single project.

It keeps permissions manageable. When a person changes teams, you change their group, and their rights follow. The guiding rule is least privilege: give each person only what they need.

## A worked example {#example}
Suppose a company wants to enforce that every production deployment has an approval step.

Here is a policy written in Rego:

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

This is how it is used.

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

Count the benefit of Policy as Code. If there are 60 pipelines, a manual review for the approval rule would take about 5 minutes each, which is 300 minutes or 5 hours, and you would have to repeat it whenever a pipeline changed. The policy checks all of them in seconds, every time.

## A common mistake {#mistake}
A common slip is to write policies in blocking mode right away, which stops teams in the middle of their work. Start in warning mode, fix the existing violations and then turn on blocking. Another slip is giving everyone the administrator role because it is easy. Use groups and least privilege.

## Key points {#key-points}
- Governance checks pipelines against rules automatically before they run.
- OPA is the open-source engine, and policies are written in Rego.
- Policy as Code keeps rules in version control, so they are precise, reviewed and testable.
- RBAC assigns roles at a scope. Give the least privilege that is needed.
