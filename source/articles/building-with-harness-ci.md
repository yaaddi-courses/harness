# Building with Harness CI: artifacts, registries and build machines

**Covers:** artifact, registry, Harness CI Cloud, VMs

## What a build produces, and where it runs {#overview}
A continuous integration pipeline takes source code and turns it into something that can be deployed. Two questions follow: what exactly comes out, where does it go and on which machines does the work happen? This article covers the output, its storage and the choice of build machines.

## artifact {#artifact}
An artifact is the packaged output of a build, such as a compiled program, a container image or a zip file, that is tested and then shipped. It is the thing that moves through the later stages. The same artifact that passed the tests should be the one that reaches production, which makes the test result mean something.

Artifacts should be named and versioned so that you can tell exactly what is running: for example `shop-api:2.4.1`, or a tag that contains the code revision. Avoid rebuilding at each stage, since two builds of the same code may not be identical.

## registry {#registry}
A registry is a storage service for built artifacts, most often container images, such as Docker Hub, that a CI pipeline uploads to and a deployment later downloads from. It is the hand-over point between CI and CD: the build pushes the image to the registry, and the deployment pulls it by name and version.

Registries can be public or private, hosted by a cloud provider or run by you. Access is controlled, so a connector with credentials is used to push and pull. Many teams keep their own private registry, and old images need to be cleaned up now and then, since storage grows with every build.

## Harness CI Cloud {#harness-ci-cloud}
Harness CI Cloud is Harness's own managed pool of virtual machines that run your builds, so you do not have to set up and maintain build machines yourself. You choose an operating system and size in the stage settings, such as Linux or macOS, and a clean machine is started for the run. When the build finishes, the machine is discarded.

The benefits are no maintenance, quick start-up and a fresh machine for each build, so leftovers from earlier runs cannot affect the result. The drawbacks are that builds run outside your network, so they cannot reach private systems without extra setup, and that you pay for the use.

## VMs {#vms}
VMs, or virtual machines, are computers made of software that run inside real ones. Build machines are usually VMs, because they are easy to create and delete, and a single physical server can host many of them. Each run gets its own VM, which isolates builds from each other.

The alternative is to run builds on machines that you manage yourself, for example in your own Kubernetes setup through a Delegate. That keeps builds inside your network, which is needed when they must reach private systems or when rules require your own hardware, but you must maintain and scale the machines.

## A worked example {#example}
A team's build stage builds an image and pushes it to a registry.

```yaml
- stage:
    name: Build
    identifier: build
    type: CI
    spec:
      platform:
        os: Linux
        arch: Amd64
      runtime:
        type: Cloud
        spec: {}
      execution:
        steps:
          - step:
              type: BuildAndPushDockerRegistry
              name: Build and push image
              identifier: build_push
              spec:
                connectorRef: dockerhub
                repo: shopteam/shop-api
                tags:
                  - <+pipeline.sequenceId>
```

Read it:

1. `runtime: Cloud` runs the stage on Harness CI Cloud, on a fresh Linux VM.
2. The step builds the image and pushes it to the registry through the `dockerhub` connector.
3. The tag is the pipeline's run number, so run 87 produces `shopteam/shop-api:87`. That image is the artifact.
4. When the stage ends, the VM is deleted.

Compare the two build locations:

| | Harness CI Cloud | Self-hosted VMs |
|---|---|---|
| Set-up effort | Almost none | Install and maintain |
| Reach private systems | Not by default | Yes, inside your network |
| Clean machine per run | Yes | Depends on your set-up |
| Cost model | Per use | Your own hardware and staff |

Suppose a team runs 400 builds a month, each taking 8 minutes. That is 3,200 build minutes, about 53 hours of machine time. A cloud service charges for those minutes. A self-hosted setup also costs them, but spread across a fixed pool that may sit idle at night, and one person may spend a few hours a month maintaining it. Neither is always cheaper: it depends on the volume and the staff.

## A common mistake {#mistake}
Rebuilding the artifact in each stage, so that what is deployed is not what was tested. Build once, push to the registry and promote the same version. Another mistake is using the `latest` tag for artifacts, which makes it unclear what is running.

## Key points {#key-points}
- An artifact is the packaged build output. Build it once and promote the same one.
- A registry stores artifacts and links CI to CD.
- Harness CI Cloud supplies managed, fresh build machines. Self-hosted VMs keep builds inside your network.
- Give artifacts clear version names so you know what is running.
