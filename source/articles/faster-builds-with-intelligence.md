# Faster builds: Test Intelligence and Cache Intelligence

**Covers:** Test Intelligence, Cache Intelligence

## Slow pipelines are a hidden tax {#overview}
When a build takes 40 minutes, developers stop waiting for it. They switch to something else, forget about it and merge on a hunch. Slow feedback also means more changes pile up before problems are found. The two biggest causes of slow builds are running tests that cannot possibly be affected and downloading the same things over and over. Harness CI has a feature aimed at each.

## Test Intelligence {#test-intelligence}
Test Intelligence is a Harness CI feature that works out which tests are actually affected by a code change and runs only those. In a large project, a change to one module usually cannot break tests of an unrelated module, so running every test wastes time.

How it works, in outline: on a first run, the tool records which tests exercise which parts of the code, by tracing the call paths. On later runs, it compares the changed files with that map and selects the tests that touch them. It can also run the tests most likely to fail first, so that a break shows up early. Support depends on the language and the test framework, so check the documentation for yours.

A safe habit is to run the full suite on a schedule, such as nightly, or before a release, as a safety net. Selection speeds up the everyday loop, and the full run guards against any case where a dependency was missed.

## Cache Intelligence {#cache-intelligence}
Cache Intelligence is a Harness CI feature that automatically reuses the dependency downloads and build outputs from earlier runs, so that each build does not start from nothing. Most builds begin by downloading libraries, which is slow, and the libraries rarely change between runs.

It saves a cache at the end of a build and restores it at the start of the next one, keyed on the files that decide the content, such as the lock file that lists the exact dependency versions. If the lock file has not changed, the cache matches and is reused. If it changed, a new cache is created. It works for common package managers, with little or no configuration.

## A worked example {#example}
A project has 2,000 tests and a build that downloads its libraries.

| Phase | Before | With both features |
|---|---|---|
| Download dependencies | 4 minutes | 20 seconds (from cache) |
| Compile | 3 minutes | 3 minutes |
| Run tests | 25 minutes (all 2,000) | 3 minutes (the 240 affected tests) |
| Total | 32 minutes | 6 minutes 20 seconds |

Do the arithmetic: 4 + 3 + 25 = 32 minutes before. After: 20 seconds + 3 minutes + 3 minutes = 6 minutes 20 seconds, which is 6.33 minutes. The saving is 25.7 minutes, about 80 percent.

Over a month, with 30 developers each making 4 changes a day for 20 working days, that is 2,400 builds. Saving 25.7 minutes on each is about 61,700 minutes, or about 1,028 hours of waiting and of build machine time. Even if developers use only a tenth of the saved time productively, that is roughly 100 hours.

A typical configuration step with Cache Intelligence is a single setting on the stage, where you turn it on, and Harness chooses the cache key from the lock file.

The safety net: a nightly pipeline with a cron trigger runs all 2,000 tests. Suppose one night it catches a failure in a test that selection had skipped, because a shared configuration file changed. That is a signal to widen the rules for that area of the code, for example by marking the file as affecting everything.

## A common mistake {#mistake}
Trusting selection completely and never running the full suite. Selection depends on the map being correct, and it can miss things such as changes to configuration or to generated code. Run everything on a schedule. Another mistake is a cache key that never changes, so that stale libraries are kept after an upgrade.

## Key points {#key-points}
- Test Intelligence runs only the tests that a change can affect, and can run likely failures first.
- Cache Intelligence reuses downloads and build output between runs, keyed on dependency files.
- Together they can cut a build from half an hour to a few minutes.
- Keep a scheduled full test run as a safety net.
