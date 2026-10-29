# CI/CD, DevOps and the problem a platform solves

**Covers:** CI/CD, CD, DevOps, OS

## Shipping software is a automated flow problem {#overview}
Writing code is only the start of delivering it. The code must be built, tested, packaged and put somewhere that users can reach. Done by hand, each action is slow and error-prone, and the whole job tends to be left until a big, stressful release day. A set of ideas and tools grew up to turn this into an automatic, routine process, and platforms such as Harness exist to run it.

## DevOps {#devops}
DevOps is a way of working in which the people who develop software and the people who run it share responsibility for both. In the older model, developers wrote code and handed it over to an operations team, who were left to keep it running, with different goals and little shared knowledge. Each side blamed the other when something broke.

In a DevOps culture, one team builds, ships and runs a product, which gives its members a reason to make it easy to release and reliable to operate. Practices that go with it are automation, small frequent changes and measurement. It is a culture first, and tools only support it.

## CI/CD {#ci-cd}
CI/CD stands for continuous integration and continuous delivery or deployment. It is the practice of automatically building, testing and shipping code whenever someone changes it.

- **Continuous integration (CI):** every change is merged into the shared code frequently, and an automatic build and test run checks it within minutes. Problems are found while the change is small and fresh in the author's mind.
- **Continuous delivery or deployment (CD):** every change that passes is prepared for release, and the release reaches users either after a person approves it or automatically.

Together they replace a quarterly release, with hundreds of changes bundled together, by many small releases a day, each easy to understand and to undo.

## CD {#cd}
CD alone is ambiguous, and has two meanings. Continuous delivery means that every change is automatically prepared so that it could be released to users at any time, while a person still decides when to release. Continuous deployment means that every change that passes the tests is released without any manual action. In the Harness product, "CD" refers to the module that handles the delivery and deployment side, which is the part after the build.

## OS {#os}
OS stands for operating system. It is the central layer of software that controls a computer's hardware and starts other programs. It matters in CI/CD because builds run on a particular operating system, such as Linux, Windows or macOS. A program built and tested on one may behave differently on another, so automated flows are often run on several, and the choice of the build machine's OS is part of the setup.

## The problem with stitching tools together {#problem}
Teams that adopt CI/CD often begin by collecting separate tools: one for builds, one for deployments, one for passwords and tokens, one for sign-offs, one for security scans. Each has its own settings, its own logins and its own way of describing a automated flow. The glue between them is scripts that nobody owns. Harness exists because of this problem. It offers the main parts as parts of one platform, so that they share one model for automated flows, users, permissions and passwords and tokens.

## A worked example {#example}
A team ships a web product with and without automation.

| | By hand | With CI/CD |
|---|---|---|
| Build and test | A developer runs them locally, when remembered | An automatic run on every change, about 6 minutes |
| Release frequency | Once a month | Several times a day |
| Changes per release | About 120 | About 3 |
| When a release breaks | Search through 120 changes | Search through 3 |
| Who runs it | An operations specialist, over a weekend | The automated flow, with an sign-off click |

Count the effect on risk. With 120 changes in a release, if each has a 1 percent chance of a problem, the chance that at least one goes wrong is 1 minus 0.99 raised to the power 120, which is about 70 percent. With 3 changes per release, it is about 3 percent. The risk per release falls from 70 to 3 percent, and finding the cause becomes much easier.

The team's operating system choice matters too: they build on Linux because their servers run Linux, and add a Windows build for the desktop client.

## A common mistake {#mistake}
Buying a CI/CD tool and expecting DevOps. If developers still hand work over to a separate operations team, the tool only automates the handover. Another mistake is to automate a broken process. Fix the process first, or automation will only make mistakes happen faster.

## Key points {#key-points}
- DevOps means that development and operations share responsibility for building and running software.
- CI/CD builds, tests and ships every change automatically. Continuous delivery waits for a person, and continuous deployment does not.
- Builds run on a particular OS, so choose it to match where the software will run.
- A platform brings the parts of CI/CD under one model, so that they do not have to be stitched together.
