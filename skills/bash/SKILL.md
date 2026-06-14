---
name: bash
description: Expert guidance for writing robust, maintainable, and "Clean Code" compliant Bash scripts. Use this when creating, refactoring, or debugging shell scripts to ensure they follow security, reliability, and engineering standards.
---

# Bash Engineering Excellence

This skill provides expert procedural guidance for writing shell scripts that are as maintainable and robust as any other code. Adhere to these principles whenever working with Bash.

## Robustness & Debugging

- Strict Mode: Always include at the start of every script to fail fast and prevent silent errors.
```bash
set -o errexit   # abort on nonzero exitstatus
set -o nounset   # abort on unbound variable
set -o pipefail  # don't hide errors within pipes
```

- Static Analysis - Always use [ShellCheck](https://www.shellcheck.net/) and fix all warnings. This is mandatory for catching common bugs.

- Syntax Validation - Run `bash -n script.sh` to check for syntax errors before running or committing.

- Debug tracing - Use `bash -x script.sh` or `set -x` for execution tracing.

## General Principles

- Clean Code - Principles of Clean Code apply to Bash. Prioritize readability.

- Long Parameters - Prefer long-form options for readability:
Good ✅
```bash
rm --recursive --force -- "${dir}"
```
Bad ❌
```bash
rm -rf -- "${dir}"
```

- Navigation - Avoid `cd ..`. Use subshells or `pushd`/`popd` to isolate directory context:
Bad ❌
```bash
cd "${foo}"
[...]
cd ..
```

Good ✅
```bash
# Subshell
(
  cd "${foo}"
  [...]
)

# Pushd/Popd
pushd "${foo}"
[...]
popd

```

- Background Processes - Use `nohup foo | cat &` if `foo` must be started from a terminal and run in the background.

## Variable Management

- Scoping - Prefer `local` variables within functions. Make global variables `readonly`.
- Form - Always use `${var}` form, not `$var`.
- Quoting - Always quote variables: `"${var}"`.
- Naming -
  - Environment (exported): `${ALL_CAPS}`
  - Local: `${lower_case}`
- Subprocesses - Be aware that variables set in subprocesses do not persist (e.g. in some piping loops). Use stdout/grep for communication.

## Functions & Abstraction

- Single Responsibility Principle (SRP) - Each function should do exactly one thing.
- Don't mix levels of abstraction.
- Positional Parameters - Assign meaningful local names to positional arguments:
```bash
foo() {
    local first_arg="${1}"
    local second_arg="${2}"
    [...]
}
```
- Readability - Extract complex tests into descriptive functions:
Bad ❌
```bash
if [ "$#" -ge "1" ] && [ "$1" = '-h' ] || [ "$1" = '--help' ] || [ "$1" = "-?" ]; then
    usage
    exit 0
fi
```

Good ✅
```bash
help_wanted() {
    [ "$#" -ge "1" ] && [ "$1" = '-h' ] || [ "$1" = '--help' ] || [ "$1" = "-?" ]
  }

  if help_wanted "$@"; then
    usage
    exit 0
  fi
```

## Substitution & Redirection

- **Command Substitution** - Always use `$(cmd)` instead of backquotes.
- **Overrides** - Prepend with `\` to override alias/builtin lookup: `\time bash -c "..."`.
- **Prefer `printf`** - Use `printf` over `echo` for portability and control.
- **Stderr** - Print errors to stderr: `printf "..." >&2`.
- **Heredocs** - Name tags meaningfully (`<<HELPMSG`) and single-quote them (`<<'MSG'`) to prevent interpolation if not needed.
- **Sudo Redirection** - Use `printf "..." | sudo tee /path/to/file > /dev/null`.

## Cleanup Code

Always implement a `finish` function with an exit trap:
```bash
finish() {
  local result=$?
  # Your cleanup code here (e.g. rm -f "${tmpfile}")
  exit ${result}
}
trap finish EXIT ERR
```

## Running

Don't attempt to run the scripts before asking the user to confirm first.
