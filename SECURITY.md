# Security and credential handling

Credenvoy handles credentials supplied by the person running it. Treat generated auth files, native OAuth bundles, optional profile documents containing literals, and refreshed agent state as secrets.

## Report a vulnerability privately

Use GitHub's **Security → Report a vulnerability** option when it is enabled on the repository. Otherwise contact a maintainer privately before sharing sensitive details. Do not post tokens, raw environment dumps, native auth files, or generated state in public issues, discussions, or CI logs.

Useful reports include a synthetic reproducer, the Task/Python/agent CLI versions, operating system, expected behavior, and sanitized observed behavior.

## Current boundaries

- Routine tests use synthetic credentials and do not contact model providers.
- `plan` reports file names, not resolved credential values.
- Generation refuses an existing output directory. Unix files/directories are created with restrictive modes.
- The launcher filters known inherited credential/provider selectors and redirects native agent state. External cloud identities and project-local agent settings can still apply.
- Bootstrap preserves later refreshes; it is not a cross-process or multi-machine OAuth synchronizer.
- OAuth client authority, scopes, account entitlement, and API billing are separate concerns. A JSON field conversion cannot manufacture them.
- Windows ACL hardening is not implemented. Windows CI validates configuration behavior, not a claim of Unix-equivalent access control.
- The repository's CI uses no model-provider secrets. The manual source-bundle workflow does not publish a package or release.

Never point the launcher at untrusted generated runtime files. Review configurations and any project-level agent hooks/settings before running an agent with access to your workspace.


## Native installation

`task install` explicitly writes selected credentials to real CLI homes; `task install:plan` previews destinations without writes or authentication commands (discovery may probe the local OpenCode version). Use environment/secret-manager inputs, stop running CLIs, and review the selected profiles. Unlike isolated generation, native installation makes persisted credentials available to ordinary CLI launches. Native files and exact sibling backups contain secrets; do not commit them. On Unix these files use mode `0600`; Windows ACLs remain the user's responsibility.

Native file replacements have backups and best-effort rollback. Copilot keychain login and OpenCode database imports are external operations and cannot be part of the same rollback transaction. Symlink targets and malformed existing files are rejected. JSONC/TOML comments and formatting are normalized during merging; original bytes remain in backups. CLI refresh locks cannot be coordinated portably, so do not install concurrently with an active session.

Profile-free discovery reads recognized credential variables from the process environment and selects only implemented native routes. It does not use existing login caches as credential sources or create gateways/token conversions. Explicit `PROVIDER`/`AUTH` selectors resolve ambiguous billing, account and shared product-key choices; malformed native subscription bundles fail before writes. Keep credentials out of Task parameters and use environment/secret-manager inputs. Existing specifications retain their precedence when no discovery selector is supplied.
