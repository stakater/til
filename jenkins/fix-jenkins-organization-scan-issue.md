# How to fix GHFileNotFoundException while scanning Jenkins organization

## Problem:

Facing `org.kohsuke.github.GHFileNotFoundException` or any other exception similar to this while scanning the jenkins github organization

## Solution

If you face `org.kohsuke.github.GHFileNotFoundException` or any other exception similar to this while scanning the jenkins github organization. 

```text
Please make sure your github credentials for that organization are correct.
```

Credentials used to scan branches and pull requests, check out sources and mark commit statuses.

Note that only "username with password" credentials are supported. Existing credentials of other kinds will be filtered out. This is because jenkins exercises GitHub API, and this last one does not support other ways of authentication.

If none is given, only the public repositories will be scanned, and commit status will not be set on GitHub.

If your organization contains private repositories, then you need to specify a credential from an user who have access to those repositories. This is done by creating a "username with password" credential where the password is GitHub personal access tokens. The necessary scope is "repo"