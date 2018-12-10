# Submit Pull Request for Helm Public Chart Repository

## Getting started

1. Clone this [stakater/charts](https://github.com/stakater/charts) repository locally. This is a fork of [helm/charts](https://github.com/helm/charts) repository.
2. Create a branch from master branch and checkout to newly created branch.
3. Pull latest changes from [helm/charts](https://github.com/helm/charts) branch.
    - Command to pull latest changes
    ```bash
    git pull git@github.com:helm/charts.git master
    ```
4. Start adding your changes.

## Improvement in existing chart

- If you are adding new resource, make sure you have optional labels and annotations.
- If you are adding new values in values.yaml, make sure you have no trailing space in values.yaml
- Make sure you have a new line at the end of each file
- When you add a comment in values.yaml, make sure there is a space between `#` and your actual comment message
- If there is an rbac, make sure it is optional
- Make sure each resource has its own yaml file, for example, service account, role, cluster-role, role-binding etc should have their separate files.
- Explain values.yaml values in readme
- Add NOTES.txt in templates directory, explaining the purpose of chart and other important notes during chart deployment.
- Bump chart version in chart.yaml
- Most important of all, **all commits should be signed off**. A single non-signed off commit can ruin the PR. since helm public chart repo does not accept non signed off commits. And amending the non signed off commits can ruin the sign off history as well.
  - Command to sign off a commit.

  ```bash
    git commit -s -am <commit-message>
  ```

## Submitting new chart

- Follow all guidelines in **Improvement in existing chart**
- Add `OWNERS` file containing the owners of chart.
    Sample `OWNERS` file
    ```yaml
    approvers:
    - faizanahmad055
    - kahootali
    - ahmadiq
    - waseem-h
    - rasheedamir
    - ahsan-storm
    reviewers:
    - faizanahmad055
    - kahootali
    - ahmadiq
    - waseem-h
    - rasheedamir
    - ahsan-storm
    ```
- Add `.helmignore` and add `OWNERS` in this.
    Sample data in `.helmignore`
    ```txt
    # OWNERS file for Kubernetes
    OWNERS
    ```
- Follow guidelines mentioned [here](https://github.com/helm/charts/blob/master/REVIEW_GUIDELINES.md) and [here](https://github.com/helm/charts/blob/master/CONTRIBUTING.md) for additional info before submitting the PR.

## Creating PR
- Add [helm/charts] and also chart name e.g. [stable/jenkins] in PR title to easily distinguish it.
- Add signoff comment in PR description e.g.
  ```text
  Signed-off-by: Faizan Ahmad
  username: faizanahmad055
  email: faizan.ahmad55@outlook.com
  ```
- Add description and purpose of your changes.
- Tick these three boxes by placing an `x` (no spaces) in `[]`
  ```text
  [x] DCO signed
  [x] Chart Version bumped
  [x] Variables are documented in the README.md
  ``` 
After creating a PR, ping the maintainers regularly.