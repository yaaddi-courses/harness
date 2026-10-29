# Harness Open Source, source control and watching runs

**Covers:** Harness Open Source, SCM, execution view

## Choosing how to run the platform {#overview}
Harness can be used as a hosted service or run on your own machines, with a smaller feature set. Which one you pick affects your cost, your control over the data and how much work you take on. Once a pipeline is running, you also need to see what it is doing. This article covers the open-source option, the tools that hold your code and the screen used to watch runs.

## SCM {#scm}
SCM, or source code management, is the family of tools, Git above all, that store your code and record every change to it. A code host such as GitHub, GitLab or Bitbucket adds collaboration features: pull requests, reviews and permissions. SCM is the starting point of every pipeline, since the code is what is built, and a change in the code is the most common event that starts a run.

## Harness Open Source {#harness-open-source}
Harness Open Source is the free, self-hostable core of Harness. It covers source code management, pipelines for CI/CD and an artifact registry. It grew from the Gitness project, which Harness took over and extended. You run it on your own server, in a single container for a start, and you keep all of the data.

Its strengths are zero licence cost, full control and a single small program to try out in an afternoon. Its limits are that the advanced modules of the commercial platform, such as governance, chaos engineering, cost management and the enterprise features, are not part of it, and that you operate, upgrade and back it up yourself.

How to choose:

| | Harness Open Source | Harness Platform (SaaS) |
|---|---|---|
| Cost | Free, plus your own servers | Subscription |
| Who runs it | You | Harness |
| Modules | Core: code, pipelines, registry | All modules |
| Data location | Your servers | Harness cloud, with options |
| Best for | Small teams, learning, strict data control | Larger organizations that want less work |

## execution view {#execution-view}
The execution view is the real-time screen in Harness where you watch a pipeline run step by step, with live logs, so that you can see exactly what is happening and where it failed. The pipeline is drawn as a flow of stages. Select a stage to see its steps, and select a step to see its output as it is produced.

Useful features include the status of each step, the time taken, the inputs the run used, the artifacts produced and buttons to retry a failed step or to re-run from a particular stage. Read from the bottom of a failed step's log upward: the real error is usually near the end.

## A worked example {#example}
A small team wants a free, self-hosted setup to try.

```bash
docker run -d -p 3000:3000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v gitness:/data \
  --name harness harness/harness
```

Steps:

1. Run the container. Harness Open Source starts on port 3000, and keeps its data in the `gitness` volume.
2. Open `localhost:3000` in a browser and create an account.
3. Create a repository, which is the SCM part. Push your code to it as to any Git server.
4. Add a pipeline file to the repository. Each push starts a run.
5. Open the run and use the execution view: stages across the top, steps on the left and the live log on the right.

Suppose that a run fails. In the execution view, the `Test` stage shows red, and its step `Run unit tests` shows a log. The last lines say `AssertionError: expected 120.0, got 100.0`. The cause is a test failure in the code, not in the pipeline. Without the view, you would have to find the log on the build machine.

Count the effort: on one small virtual machine with 2 CPUs and 4 GB of memory, the whole setup is a few minutes' work and costs the price of the machine, perhaps 20 dollars a month. A commercial subscription for the same team may cost more, but includes the operation, the upgrades and the other modules.

If the team later needs approvals, governance and cost control, it can move to the platform. The pipeline ideas, stages and steps, carry over.

## A common mistake {#mistake}
Choosing Open Source to save money and then not budgeting for the work of running it: backups, updates and security patches. Another mistake is mounting the Docker socket into a container without understanding that it gives that container control of the host's containers. Do it only on a machine that you trust.

## Key points {#key-points}
- SCM tools store code and its history, and are where most pipelines begin.
- Harness Open Source is a free, self-hosted core with code hosting, pipelines and a registry.
- The commercial platform adds the advanced modules and removes the operating work.
- The execution view shows a run live. Read a failing step's log from the end.
