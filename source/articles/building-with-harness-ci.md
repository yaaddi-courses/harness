# Building with Harness CI: artifacts, registries and build machines

**Covers:** artifact, registry, Harness CI Cloud, VMs

## What a build produces, and where it runs {#overview}
A continuous integration pipeline takes your source code and turns it into something that can be deployed. Three questions follow. What exactly comes out? Where does it go? And on which machines does the work happen? In this article we answer them in that order: the output, its storage and the choice of build machines.

## artifact {#artifact}
An artifact is the packaged output of a build, such as a compiled program, a container image or a zip file. It is tested and then shipped, and it is the thing that moves through the later stages. The artifact that passed the tests should be the one that reaches production, because only then does the test result mean something.

Name and version your artifacts so that you can tell exactly what is running, for example `shop-api:2.4.1`, or a tag that contains the code revision. Avoid rebuilding at each stage, since two builds of the same code may not be identical.

## registry {#registry}
A registry is a storage service for built artifacts, most often container images, such as Docker Hub. A CI pipeline uploads to it, and a deployment later downloads from it. It is the hand-over point between CI and CD: the build pushes the image to the registry, and the deployment pulls it by name and version.

Registries can be public or private, hosted by a cloud provider or run by you. Access is controlled, so you use a connector with credentials to push and pull. Many teams keep their own private registry, and they clean out old images now and then, because storage grows with every build.

## Harness CI Cloud {#harness-ci-cloud}
Harness CI Cloud is Harness's own managed pool of virtual machines for running your builds, so you do not set up and maintain build machines yourself. In the stage settings you choose an operating system and a size, such as Linux or macOS, and a clean machine is started for the run. When the build finishes, the machine is discarded.

The benefits are no maintenance, quick start-up, and a fresh machine for every build, so leftovers from earlier runs cannot affect the result. There are two drawbacks. Builds run outside your network, so they cannot reach private systems without extra setup. And you pay for the use.

## VMs {#vms}
VMs, or virtual machines, are computers made of software that run inside real ones. Build machines are usually VMs, because they are easy to create and delete, and one physical server can host many of them. Each run gets its own VM, which keeps builds isolated from each other.

The alternative is to run builds on machines you manage yourself, for example in your own Kubernetes setup through a Delegate. That keeps the builds inside your network, which you need when they must reach private systems or when rules require your own hardware. In return, you must maintain and scale the machines.

## A worked example {#example}
Let's look at a team's build stage that builds an image and pushes it to a registry.

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

Here is how to read it.

1. `runtime: Cloud` runs the stage on Harness CI Cloud, on a fresh Linux VM.
2. The step builds the image and pushes it to the registry through the `dockerhub` connector.
3. The tag is the pipeline's run number, so run 87 produces `shopteam/shop-api:87`. That image is the artifact.
4. When the stage ends, the VM is deleted.

Now compare the two build locations:

| | Harness CI Cloud | Self-hosted VMs |
|---|---|---|
| Set-up effort | Almost none | Install and maintain |
| Reach private systems | Not by default | Yes, inside your network |
| Clean machine per run | Yes | Depends on your set-up |
| Cost model | Per use | Your own hardware and staff |

Suppose a team runs 400 builds a month, each taking 8 minutes. That is 400 × 8 = 3,200 build minutes, about 53 hours of machine time. A cloud service charges for those minutes. A self-hosted setup costs money too, but spread across a fixed pool that may sit idle at night, and one person may spend a few hours a month maintaining it. Neither is always cheaper. It depends on the volume and on your staff.

## A common mistake {#mistake}
A common slip is to rebuild the artifact in each stage, so that what is deployed is not what was tested. Build once, push to the registry and promote the same version. Another slip is to use the `latest` tag for artifacts, which makes it unclear what is running.

## Key points {#key-points}
- An artifact is the packaged build output. Build it once and promote the same one.
- A registry stores artifacts and links CI to CD.
- Harness CI Cloud supplies managed, fresh build machines. Self-hosted VMs keep builds inside your network.
- Give artifacts clear version names so you know what is running.
