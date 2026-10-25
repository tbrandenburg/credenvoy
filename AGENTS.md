# Working on Credenvoy

Credenvoy is a self-contained collection of go-task Taskfiles that runs a Python standard-library configuration compiler. Preserve that distribution model: copying the root Taskfile and `tasks/` must be sufficient to use and test the project.

## Repository map

- `Taskfile.yml`: public command names and orchestration. Keep it small.
- `tasks/runtime.yml`: readable Python literal blocks named `RENDERER_SOURCE`, `COMPILER_SOURCE`, `LAUNCHER_SOURCE`, `INSTALLER_SOURCE`, and `BOOTSTRAP_SOURCE`. The private `_dispatch` task assembles modules in memory.
- `tasks/catalog.yml`: versioned provider registry, known env names, and examples. Keep provider IDs, protocols, auth modes, aliases, and source links explicit.
- `tasks/tests.yml`: readable Python test blocks. Tests execute in the same in-memory module namespace as the runtime.
- `README.md`: user guide and honest support claims.
- `EXAMPLES.md`: generated Markdown walkthrough; refresh with `task examples:write`, check with `task examples:check`. Synthetic inputs stay embedded in `CATALOG_SHOWCASE`; no native credential files are committed.
- `.github/workflows/`: credential-free validation and portable source-bundle creation.

There is intentionally no `src/` package, external JSON registry, fixture directory, generated matrix checked into Git, or runtime pip dependency.

## Commands

Requires Python 3.11+ and Task 3.54+:

```sh
task install:plan      # preview native user paths; implementation work must use synthetic homes
task check             # syntax/invariants, all tests, offline generation smoke
task test              # focused test suite
task lint              # embedded runtime syntax and registry invariants
task matrix            # machine-readable current routes
task matrix:markdown   # human-readable current routes
task smoke             # offline local configuration generation
task examples:write    # refresh documentation after output/fixture changes
task examples:check    # check generated Markdown against the current compiler
```

Use `PYTHON=python3` when needed. Validate from a temporary copy containing only `Taskfile.yml` and `tasks/` after changes that affect paths, imports, examples, or bootstrap. CI performs this check on Linux, macOS and Windows. No provider login is required for routine development.

## Editing embedded Python

1. Read the relevant YAML literal block before changing it. Maintain YAML indentation and standard Python formatting; do not minify the code or hide it in base64.
2. Keep the runtime Python-only and standard-library-only. Tests must not import a local package or read a companion fixture from the repository.
3. `RENDERER_SOURCE` owns JSON/TOML serialization and private staging. `COMPILER_SOURCE` owns profile validation and route selection. `LAUNCHER_SOURCE` owns environment isolation and bootstrap. `INSTALLER_SOURCE` owns explicit native installation, destination resolution, merge/replace rules, backups and command-based login/import. `BOOTSTRAP_SOURCE` owns module assembly and command dispatch.
4. The dispatcher injects public catalog data into module globals. It does not inject credential values into Python source. Secrets are resolved from the process environment by the running compiler.
5. Preserve UTF-8 output and platform-correct paths. Check Unix mode assertions only on Unix; do not imply that chmod sets Windows ACLs.
6. Keep public error text free of credential values. Catch failures without echoing raw input JSON, subprocess output, or environment dumps.

`EXAMPLES.md` is an explicit documentation exception to the generated-auth-file exclusion: it contains only synthetic values in Markdown code blocks. `task check` must still work without this document in a portable Taskfile-only copy.

If you temporarily extract code for formatting, keep extraction output outside the tracked project and re-embed it before delivery. `task export` is an optional user-facing standalone-engine export, not a second source of truth.

## Adding a provider or authentication method

- Add an explicit entry in `CATALOG_REGISTRY` with provider/product/region identity, auth modes, endpoint protocol, conventional env names, and primary-source links where available.
- Verify upstream documentation or source for the exact CLI release/format. A catalog listing is not evidence of a usable coding model or supported subscription billing.
- Do not treat every models.dev env hint as an interchangeable API key. Cloud identifiers, credential components, and alternate auth modes need separate validation.
- Select a route in the compiler. Unsupported combinations must fail; a compatible gateway must be an explicit endpoint with an explicit credential source.
- Add meaningful assertions to `TEST_COMPILER`, `TEST_RENDERER`, or `TEST_LAUNCHER` for the behavior and its failure cases. The matrix test must continue to exercise every registry combination.
- Update README examples and counts if coverage changes. `task matrix` is the authoritative machine-readable matrix.

## Credential and subscription boundaries

- Use synthetic credentials in tests. Do not inspect developer logins, real auth caches, cloud key files, keychains, or secret-manager output for routine implementation work.
- Never commit `.env`, `profiles.json`, generated auth/config files, exports, logs, or live credentials. Keep `.gitignore` aligned with new output paths.
- API keys, upstream OAuth credentials, inference session tokens, bearer tokens, and native auth bundles are separate types. Do not silently replace a subscription route with API billing.
- Native bundles preserve unknown fields. Token translation must preserve known issuer/client provenance, method, account metadata, and genuine expiry; it cannot create scopes or change token authority.
- Installation is explicitly authorized to replace selected native credentials. Test it only against temporary homes and mocked native commands; never invoke install against the developer home during implementation. Preserve unrelated providers/settings, back up originals privately, suppress third-party output, and fail before writes on unsupported persistent mappings.
- Do not overwrite refreshed credentials during routine launches. Bootstrap is distinct from rotation and cross-machine synchronization.
- Keep new output directories private, refuse overwrites, and avoid shared servers/databases during bootstrap. Atomic writes alone do not coordinate with an external CLI's token refresh lock.
- Preserve launch-time credential/provider-selector cleanup and explicit endpoint selection. Never interpolate secret values into Taskfile command text.

## CI and publication

- CI is offline with respect to model providers and OAuth. Do not add provider secrets or real inference smoke tests to ordinary pull-request workflows.
- Pin third-party actions to immutable commit SHAs with a version comment. Workflow permissions should remain `contents: read` unless a specific workflow genuinely needs more.
- Keep build artifacts limited to tracked source files. The packaging workflow uploads a source bundle; it does not publish a package or create a GitHub release.
- A request to prepare a repository does not authorize pushing it or publishing a release. Do that only when the user explicitly requests it.
- Never invent a GitHub owner, repository URL, maintainer email, passing hosted CI result, live-integration validation result, or award claim.

## Completion checklist

Run the relevant checks once after the final code change. Report what changed, which checks actually ran, and any remaining limitations. Separate synthetic configuration coverage from live authentication, model entitlement, billing, or platform-isolation guarantees. Keep the final repository free of superseded reports, prototype modules, cache directories, and generated outputs.

