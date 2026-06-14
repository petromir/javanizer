---
name: git
description: Expert guidance for using the Git. Use when you commit messages or manage issues, pull requests, and metadata of GitHub repositories
---

# GitHub CLI (gh) Excellence

## Prerequisites

### Token

Before proceeding with any command, check if the `GH_TOKEN` environment variable is set by executing
```shell
[ -n "${GH_TOKEN+x}" ] && echo "Variable set" || echo "Variable MISSING"
```

If the token is not set, **ask the user** to go at https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token and create one with the following scopes:
- **Metadata**: Read access (required for basic repository info).
- **Issues**: Read and Write access.
- **Pull Requests**: Read and Write access.

## GitHub

Refer to [github-cli](./references/github-cli.md) procedural guidance for using the GitHub CLI to interact with GitHub resources efficiently and securely.
