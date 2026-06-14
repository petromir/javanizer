## Conventional Commits 1.0.0

## Summary

The Conventional Commits specification is a lightweight convention on top of commit messages. This convention 
dovetails with Semantic Versioning, by describing the features, fixes, and breaking changes made in commit messages.

The commit message should be structured as follows:
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Requirement Levels
- `MUST`, `REQUIRED`, `SHALL` -> definition is an absolute requirement of the specification.
- `MUST NOT`, `SHALL NOT` -> definition is an absolute prohibition of the specification.
- `SHOULD`, `RECOMMENDED`, mean that there may exist valid reasons in particular circumstances to ignore a particular item
- `SHOULD NOT`, `NOT RECOMMENDED` mean that there may exist valid reasons in particular circumstances when the particular behavior is acceptable or even useful
- `MAY`, `OPTIONAL`, mean that an item is truly optional.

## Specification

### Header
1. Commits MUST be prefixed with a type, which consists of a noun, `feat`, `fix`, etc., followed by the OPTIONAL scope, OPTIONAL `!`, and REQUIRED terminal colon and space.
2. The type `feat` MUST be used when a commit adds a new feature to your codebase.
3. The type `fix` MUST be used when a commit represents a bug fix in your codebase.
4. A scope MAY be provided after a type. A scope MUST consist of a noun describing a section of the codebase surrounded by parenthesis, e.g., `feat(parser): add ability to parse arrays`.
5. A description MUST immediately follow the colon and space after the type/scope prefix. The description is a short summary of the code changes, e.g., `fix: array parsing issue`.
6. Types other than `feat` and `fix` MAY be used in your commit messages, e.g., `build:`, `chore:`, `ci:`, `docs:`,`style:`, `refactor:`, `perf:`, `test:`, and others. Additional types are not mandated by the Conventional Commits specification and have no implicit effect in Semantic Versioning (unless they include a BREAKING CHANGE).
7. `fix` type commits should be translated to `PATCH` releases. `feat` type commits should be translated to `MINOR` releases. Commits with `BREAKING CHANGE` in the commits, regardless of type, should be translated to `MAJOR` releases.
8. If included in the type/scope prefix, breaking changes MUST be indicated by a `!` immediately before the `:`. If `!` is used, `BREAKING CHANGE:` MAY be omitted from the footer section, and the commit description SHALL be used to describe the breaking change.

### Body

1. A longer commit body MAY be provided after the short description, providing additional contextual information about 
the code changes. The body MUST begin one blank line after the description.
2. A commit body is free-form and MAY consist of any number of newline-separated paragraphs.

### Footer

1. One or more footers MAY be provided one blank line after the body. Each footer MUST consist of a word token, 
   followed by either a `:<space>` or `<space>#` separator, followed by a string value
2. A footer’s token MUST use `-` in place of whitespace characters, e.g., `Acked-by` (this helps differentiate the 
   footer section from a multi-paragraph body). An exception is made for `BREAKING CHANGE`, which MAY also be used as a token.
3. A footer’s value MAY contain spaces and newlines, and parsing MUST terminate when the next valid footer 
   token/separator pair is observed.
4. Breaking changes MUST be indicated as:
   - uppercase text `BREAKING CHANGE:` entry in the footer, followed by a colon, space, and description, e.g., 
     `BREAKING CHANGE: environment variables now take precedence over config files`.
   - or by appending a `!` after the type/scope. A BREAKING CHANGE can be part of commits of any *type*.
5. `BREAKING-CHANGE` MUST be synonymous with `BREAKING CHANGE`, when used as a token in a footer.

## Generic
1. Conventional Commits does not make an explicit effort to define revert behavior. Instead, we leave it to tooling
    authors to use the flexibility of `types` and `footers` to develop their logic for handling reverts.
2. The units of information that make up Conventional Commits MUST NOT be treated as case-sensitive by implementors, 
   with the exception of BREAKING CHANGE which MUST be uppercase.

## Examples

### Commit message with description and breaking change footer
```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

### Commit a message with! to draw attention to breaking change

```
feat!: send an email to the customer when a product is shipped
```

### Commit a message with scope and! to draw attention to breaking change

```
feat(api)!: send an email to the customer when a product is shipped
```

### Commit a message with both! and BREAKING CHANGE footer

```
feat!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

### Commit a message without a body and different type

```
docs: correct spelling of CHANGELOG
```

### Commit message with scope

```
feat(lang): add Polish language
```

### Commit a message with a multi-paragraph body and multiple footers

```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```

### Revert with commit SHAs footer being reverted

```
revert: let us never again speak of the noodle incident

Refs: 676104e, a215868
```