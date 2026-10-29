# Pipelines, stages, steps and modules

**Covers:** pipeline, stage, step, modules, Feature Flags

## The structure of everything in Harness {#overview}
Harness organizes work in layers, from the whole delivery process down to a single action. If you learn the layers early, the rest of the platform becomes much easier to read, because every screen and every file uses them. The platform itself is divided into modules, and each module is aimed at one job. Let's go from the outside in.

## pipeline {#pipeline}
A pipeline is the top-level workflow in Harness. It is the complete sequence of stages that fetches your code, builds it, tests it and deploys it, and everything else plugs into it. When people talk about "running Harness", they mean running a pipeline.

You define a pipeline as configuration, either in the visual editor or in a text file, and it has a name, an identifier and some inputs. Because the definition is text, you can keep it in version control, review changes to it and copy it.

## stage {#stage}
A stage is one phase of a pipeline, such as build, test or deploy, and it groups a set of related steps. Stages usually run in order, and a failing stage stops the later ones unless you choose otherwise. Each stage has a type that decides what it can do: a build stage belongs to continuous integration, and a deploy stage to continuous delivery.

A stage also carries settings such as where it runs, and which application and which target place it uses. If you split a pipeline into well-named stages, it is easy to see where a run failed.

## step {#step}
A step is the smallest unit of work in a pipeline: one specific action, such as running a script, running a group of tests or applying a deployment. Steps inside a stage run in order, and some step types can run in parallel. Each step produces a result, success or failure, and output that appears in the logs.

Harness has many built-in step types, such as running shell commands, building and pushing an image or sending a notification. You can also add steps from the community or write your own.

## modules {#modules}
In Harness, modules are separate product areas, each built for one job, and they share the platform's common features such as users, permissions, passwords and pipelines. Examples include Continuous Integration (CI) for building and testing, Continuous Delivery (CD) for deploying, Feature Flags, and others for security, reliability, cost and the developer experience. You adopt the ones you need and add others later.

## Feature Flags {#feature-flags}
Feature Flags is the module that lets you turn a piece of functionality on or off while the software is running, either for everyone or for a chosen subset of users, and without releasing new code. It separates deploying code from releasing a feature. The code can sit in production switched off, be turned on gradually, and be switched off again instantly if there is a problem.

## A worked example {#example}
A small pipeline written as text shows the layers:

```yaml
pipeline:
  name: Shop app
  identifier: shop_service
  stages:
    - stage:
        name: Build
        identifier: build
        type: CI
        spec:
          execution:
            steps:
              - step:
                  type: Run
                  name: Run unit tests
                  identifier: unit_tests
                  spec:
                    command: npm test
    - stage:
        name: Deploy to staging
        identifier: deploy_staging
        type: Deployment
```

Let's read it from the outside in.

1. The `pipeline` is the whole workflow, named "Shop app".
2. It has two stages: Build, of type CI, and Deploy to staging, of type Deployment, which belongs to CD.
3. The Build stage holds one step, a `Run` step that executes `npm test`.
4. If the test step fails, the Build stage fails, and the deploy stage does not start.

This layout mirrors the modules. The first stage uses the CI module, and the second uses the CD module. Both live in one pipeline, with one set of logs and permissions, instead of two tools joined by a script.

Now count the parts: 1 pipeline, 2 stages and 1 shown step. A real pipeline might have 3 stages and 20 steps. When a run fails at "Run unit tests", you know within seconds that the failure is in the Build stage, and which step caused it.

A feature flag would sit beside this. The new checkout code is deployed in the Deploy stage with its flag off. After deployment, you switch the flag on for 5 percent of users, with no pipeline run at all.

## A common mistake {#mistake}
A common slip is to build one giant stage with 40 steps. When it fails, you have to read a long log, and nothing can be reused. Split it into stages with clear purposes. Another slip is to treat a feature flag as the same thing as a deployment. The flag decides who sees the feature, and the pipeline decides what code is running.

## Key points {#key-points}
- A pipeline is the whole workflow. It is made of stages, and each stage is made of steps.
- Pipelines are defined as text, so you can store and review them like code.
- Modules are product areas such as CI, CD and Feature Flags that share one platform.
- A feature flag turns a feature on or off at runtime, separately from deploying code.
