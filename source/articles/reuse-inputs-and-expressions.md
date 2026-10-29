# Reusing pipelines: templates, input sets and expressions

**Covers:** template, YAML, Input Set, overlay, expression, variables

## Define it once {#overview}
A growing company soon has dozens of pipelines that are nearly the same: build, test, scan, deploy. Copying and editing each one leads to drift, where every copy differs a little and a fix must be made many times. Harness gives you three tools to avoid this: templates for shared structure, input sets for shared values and expressions for values that are worked out at run time.

## YAML {#yaml}
YAML is a plain-text file format for configuration that uses indentation instead of brackets. Harness pipelines, templates and input sets can all be written in it. A list item begins with a dash, and nesting is shown by indenting with spaces. YAML is easy to read, but a wrong indent can silently change the meaning, and tabs are not allowed. Because it is text, it can be stored in Git, compared and reviewed.

## template {#template}
A template is a piece of pipeline configuration, a step, a stage or a whole pipeline, that you define once and then reuse across many pipelines. If 40 services all use the same "build and scan" stage, you write it as a stage template, and each pipeline refers to it.

When the template is improved, for example when a security scan is added, every pipeline that uses it gets the change, as long as it follows the latest version. Templates can be versioned, so that you can test a new version before adopting it widely. They can also take inputs, so the same template serves different services.

## variables {#variables}
Variables are named placeholders in a pipeline or template that can be given a different value for each run or each service. A variable might hold the target namespace or the number of replicas. Using them keeps the logic fixed and lets the details change. A variable can be fixed in the definition, or marked as a runtime input, which the user supplies at the time of the run, written `<+input>`.

## expression {#expression}
A Harness expression is a placeholder written like `<+pipeline.name>` that is replaced with a real value when the pipeline runs. Expressions can read information about the run, such as the pipeline name, the service, the artifact tag or the output of an earlier step, for example `<+pipeline.stages.build.spec.execution.steps.test.output.outputVariables.coverage>`.

They are what let one definition adapt to each run: tag the image with `<+pipeline.sequenceId>`, deploy `<+artifacts.primary.tag>` and print `<+env.name>`. The value is only known at run time, so errors in an expression appear only when the step runs.

## Input Set {#input-set}
An Input Set is a saved, named group of the input values for a pipeline, for example all the settings needed to deploy to staging. Instead of typing ten values every time, the user chooses the input set. They can be stored with the pipeline in Git.

## overlay {#overlay}
An overlay combines several input sets into one run configuration. For example, a `staging-network` input set and a `small-instance` input set can be layered, and later ones take priority when they set the same value. It lets you build many combinations from a few small, reusable pieces.

## A worked example {#example}
A stage template for building and scanning, with a runtime input for the service name:

```yaml
template:
  name: Build and scan
  identifier: build_and_scan
  versionLabel: v2
  type: Stage
  spec:
    type: CI
    spec:
      execution:
        steps:
          - step:
              type: Run
              name: Build
              identifier: build
              spec:
                command: ./build.sh <+stage.variables.service>
    variables:
      - name: service
        type: String
        value: <+input>
```

An input set for one service:

```yaml
inputSet:
  name: shop-api staging
  identifier: shop_api_staging
  pipeline:
    identifier: build_deploy
    stages:
      - stage:
          identifier: build
          template:
            templateInputs:
              variables:
                - name: service
                  value: shop-api
```

How it works:

1. The template defines the stage once. Its `service` variable is a runtime input.
2. The input set `shop-api staging` supplies `shop-api`, so a run needs one click, not ten fields.
3. At run time, the expression `<+stage.variables.service>` becomes `shop-api`, and the step runs `./build.sh shop-api`.

Count the saving. With 40 services and no template, adding a new scan step means editing 40 pipelines. With a template, it means 1 edit and a new version. If the stage has 6 lines of settings that were wrong in 5 of 40 copies, the template removes the 5 mistakes and the chance of the next one.

## A common mistake {#mistake}
Making a template so flexible that it has 30 inputs, which nobody understands. Keep templates focused, and give inputs sensible defaults. Another mistake is changing a widely used template in place without a version, which changes every pipeline at once, including ones that are not ready.

## Key points {#key-points}
- Templates define a step, stage or pipeline once for many pipelines. Version them.
- Variables and runtime inputs let one definition serve many cases.
- Expressions such as `<+pipeline.name>` are filled with real values at run time.
- Input sets save values, and overlays combine them. Everything is plain YAML that can live in Git.
