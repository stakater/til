# Jenkins CI skip plugin
Jenkin's `CI Skip` plugin helps to skip building a pipeline through commit messages.

## Use Case
Most of the times we want to update documentation for a project but the push in the repository
triggers a new build and bumps it to the next version. We do not want to create a new version
just for a slight change in documentation.

## How it works
Jenkins is based on works by changeset, so if there is changeset from before build and commit includes [ci skip], then build is skipped as NOT_BUILT. If there is no changeset, it will be build.

## Installation
1. Install CI Skip using Jenkins Plugin Manager.
2. Enable ci-skip option in the Job description for that specific job.

## How to Use
Write `[ci skip]` tag in the commit messages like following to skip building for documentation update
```
git commit -m 'documentation update [ci skip]'
```