## Context Verification
Always verify the current repository context with `gh repo view` before performing destructive or state-changing operations.

## Supported Commands

### Issue Management

| Description      | Command                                                                     | Variable                                                                                       |
|------------------|-----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| List Issues      | `gh issue list --label "<type>" --assignee "@me"`                           | `<type>` provided by the user                                                                  |
| View Issue       | `gh issue view <number>`                                                    | `<number>` provided by the user                                                                |
| Create Issue     | `gh issue create --title "<title>" --body "<body>" --label "priority:high"` | `<title>` short summary of non-pushed commits; `<body>` detailed summary of non-pushed commits |
| Comment on Issue | `gh issue comment <number> --body "Add context..."`                         | `<number>` provided by the user                                                                |


### Pull Request Management

| Description  | Command                                                  | Variable                                                                                       |
|--------------|----------------------------------------------------------|------------------------------------------------------------------------------------------------|
| List PRs     | `gh pr list --state open`                                | None                                                                                           |
| Create PR    | `gh pr create --title "<title>" --body "<body>" --draft` | `<title>` short summary of non-pushed commits; `<body>` detailed summary of non-pushed commits |
| Check Status | `gh pr status`                                           | None                                                                                           |
| Review PR    | `gh pr review <number> --approve --body "<body>"`        | `<number>` provided by the user; `<body>` - **ask the user**                                   |
| Merge PR     | `gh pr merge <number> --squash --delete-branch`          | `<number>` provided by the user                                                                |


## Best Practices
- **Scripting**: Use `--json` and `--jq` for robust parsing of command output in scripts.
  ```bash
  gh pr list --json number,title --jq '.[] | "#\(.number) \(.title)"'
  ```
- **Non-Interactive**: Never use `-w` or `--web` options; I prefer direct CLI output or JSON for processing.
