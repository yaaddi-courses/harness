# Harness Open Source, source control and watching runs

**Covers:** Harness Open Source, SCM, execution view

## Choosing how to run the platform {#overview}
You can use Harness as a hosted service, or run a smaller feature set on your own machines. Which one you pick affects your cost, your control over the data and how much work you take on. And once a pipeline is running, you need to see what it is doing. In this article we cover the open-source option, the tools that hold your code, and the screen you use to watch runs.

## SCM {#scm}
SCM, or source code management, is the family of tools, Git above all, that store your code and record every change to it. A code host such as GitHub, GitLab or Bitbucket adds collaboration features: pull requests, reviews and permissions. SCM is the starting point of every pipeline, because the code is what gets built, and a change in the code is the most common event that starts a run.

## Harness Open Source {#harness-open-source}
Harness Open Source is the free, self-hostable core of Harness. It covers source code management, pipelines for CI/CD and an artifact registry. It grew from the Gitness project, which Harness took over and extended. You run it on your own server, in a single container to start with, and you keep all of the data.

Its strengths are zero licence cost, full control, and a single small program that you can try out in an afternoon. It also has limits. The advanced modules of the commercial platform, such as governance, chaos engineering, cost management and the enterprise features, are not part of it, and you operate, upgrade and back it up yourself.

This table helps with the choice:

| | Harness Open Source | Harness Platform (SaaS) |
|---|---|---|
| Cost | Free, plus your own servers | Subscription |
| Who runs it | You | Harness |
| Modules | Core: code, pipelines, registry | All modules |
| Data location | Your servers | Harness cloud, with options |
| Best for | Small teams, learning, strict data control | Larger organizations that want less work |

## execution view {#execution-view}
The execution view is the real-time screen in Harness where you watch a pipeline run step by step, with live logs, so you can see exactly what is happening and where it failed. The pipeline is drawn as a flow of stages. Select a stage to see its steps, and select a step to see its output as it is produced.

It also gives you the status of each step, the time taken, the inputs the run used, the artifacts produced, and buttons to retry a failed step or to re-run from a particular stage. When a step fails, read its log from the bottom upward, because the real error is usually near the end.

## A worked example {#example}
Imagine a small team that wants a free, self-hosted setup to try.

```bash
docker run -d -p 3000:3000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v gitness:/data \
  --name harness harness/harness
```

Here are the steps.

1. Run the container. Harness Open Source starts on port 3000, and keeps its data in the `gitness` volume.
2. Open `localhost:3000` in a browser and create an account.
3. Create a repository, which is the SCM part. Push your code to it as you would to any Git server.
4. Add a pipeline file to the repository. Each push starts a run.
5. Open the run and use the execution view: stages across the top, steps on the left and the live log on the right.

Suppose a run fails. In the execution view, the `Test` stage shows red, and its step `Run unit tests` shows a log. The last lines say `AssertionError: expected 120.0, got 100.0`. The cause is a test failure in the code, not in the pipeline. Without the view, you would have to find the log on the build machine.

Now think about the effort. On one small virtual machine with 2 CPUs and 4 GB of memory, the whole setup takes a few minutes and costs the price of the machine, perhaps 20 dollars a month. A commercial subscription for the same team may cost more, but it includes the operation, the upgrades and the other modules.

If the team later needs approvals, governance and cost control, it can move to the platform. The pipeline ideas, stages and steps, carry over.

## A common mistake {#mistake}
A common slip is to choose Open Source to save money and then not budget for the work of running it: backups, updates and security patches. Another is to mount the Docker socket into a container without understanding that it gives that container control of the host's containers. Do it only on a machine you trust.

## Key points {#key-points}
- SCM tools store code and its history, and are where most pipelines begin.
- Harness Open Source is a free, self-hosted core with code hosting, pipelines and a registry.
- The commercial platform adds the advanced modules and removes the operating work.
- The execution view shows a run live. Read a failing step's log from the end.
