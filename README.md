<div align="center">

# Credenvoy

### Your credentials, delivered to every agent.

**One profile. Native configuration. An isolated place to run.**

[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Task 3.54+](https://img.shields.io/badge/Task-3.54%2B-29BEB0?logo=task&logoColor=white)](https://taskfile.dev/)
[![Runtime dependencies: stdlib](https://img.shields.io/badge/runtime-Python_standard_library-596579)](#quick-start)
[![License: MIT](https://img.shields.io/badge/license-MIT-7C3AED)](LICENSE)

[Quick start](#quick-start) · [Generated examples](EXAMPLES.md) · [Commands](#commands) · [Subscriptions](#subscription-access) · [Compatibility](#compatibility) · [Contributing](AGENTS.md)

</div>

---

Claude Code wants one shape. OpenCode wants another. Pi, Codex, and Copilot have their own rules—and an API key is a different credential from a subscription login.

**Credenvoy turns explicit profiles and environment variables into the configuration each CLI expects.** Run `task install` once, then launch your agent CLIs normally. Credentials go to their native user locations; private generation and isolated launching are also available.

Everything that runs Credenvoy lives in **four Taskfiles**. No Python package to install, registry to download, or hidden network bootstrap.

- **Six adapters:** Claude Code, OpenCode V1, OpenCode V2, Pi, Codex, and GitHub Copilot CLI.
- **44 provider/service profiles:** API keys, cloud identity, subscription bundles, and local models.
- **288 explicit routes:** every configured combination gets a generation route, a gateway requirement, or a clear rejection.
- **One-call native setup:** `install` compiles your selected profiles directly into normal CLI homes, with private backups and merging.
- **Review before use:** `install:plan` previews native destinations without writes, CLI commands, or credential values; `plan` previews isolated generation.
- **Keep sessions separate:** the launcher isolates agent state and preserves credentials refreshed after bootstrap.

## Quick start

Install [go-task](https://taskfile.dev/docs/installation) **3.54+** and [Python](https://www.python.org/downloads/) **3.11+**. From a checkout or extracted archive:

```sh
task --list
task check
```

If your Python executable is named `python3`, use `task check PYTHON=python3`. The same override works with other commands.

### Authenticate your CLIs in one call

Define the CLI/provider combinations in `profiles.json` (see [profile examples](#a-profile-is-deliberately-small)), then supply their credentials through your existing environment or secret manager:

```sh
task install:plan       # optional preview of the real destinations
task install            # compiles and installs all selected profiles

# Launch normally, from any directory. No Credenvoy launcher is needed.
codex
opencode
pi
claude
copilot
```

Only launch the CLIs you selected in your profiles. `install` reads the specification and environment directly: **no `task generate`, generated directory, shell sourcing, or profile wrapper is required.** To install one profile from a larger specification:

```sh
AGENTCFG_PROFILE=codex-chatgpt task install
```

Stop running CLI sessions before installation. Installing is an explicit credential replacement operation; the CLIs can refresh the installed credentials afterward. Re-running installation with a stale OAuth bundle can replace newer tokens. Routine CLI launches never reinstall snapshots.

| Adapter | Native installation target / operation |
|---|---|
| OpenCode V1 | `~/.local/share/opencode/auth.json`; merge config into `~/.config/opencode/opencode.json` or an existing `opencode.jsonc` |
| OpenCode V2 | Native `opencode auth import --standalone` into the normal local database; custom-provider config in the normal OpenCode config directory |
| Codex | `~/.codex/auth.json` and `~/.codex/config.toml` |
| Pi | `~/.pi/agent/auth.json`, `settings.json`, and `models.json`, as applicable |
| Claude Code | `~/.claude/settings.json` and a supplied native `.credentials.json` bundle |
| GitHub Copilot | Native `copilot login --with-token` via stdin; the CLI chooses its credential store. Model preference in `~/.copilot/settings.json` |

Existing `CODEX_HOME`, `CLAUDE_CONFIG_DIR`, `PI_CODING_AGENT_DIR`, `COPILOT_HOME`, `XDG_DATA_HOME`, `XDG_CONFIG_HOME`, `OPENCODE_CONFIG`, and `OPENCODE_DB` overrides are honored. Overrides must be absolute; installer targets containing symlinks are rejected. Point an override at the physical directory if you manage dotfiles through symlinks. File-only routes can be configured before installing the CLI; native command routes require the corresponding CLI and flags. An installed OpenCode binary must match the selected major version; V1 and V2 cannot be mixed in one native installation.

**Merging and backups.** Unrelated provider credentials and settings survive. Selected provider credentials are replaced as complete entries, so old OAuth fields cannot contaminate API-key credentials. Codex has one native identity file, so its supplied auth bundle replaces that file as a whole. A changed existing file gets an exact private sibling backup named `<filename>.credenvoy-backup-<unique-id>`. Existing OpenCode V2 databases get a SQLite snapshot that includes committed WAL data. No-op file installations create no extra backups. New directories use mode `0700` and files/backups `0600` on Unix; these modes do not establish Windows ACLs.

JSONC comments and trailing commas are accepted. Modified configuration is serialized as JSON/TOML: **values are preserved, comments and layout are normalized**. Exact originals remain in backups. When several profiles configure one CLI, providers accumulate and the last profile's model/default wins; conflicting profiles for the same CLI/provider fail before writes. Use `AGENTCFG_PROFILE` to select a single account/default.

**Installation boundaries.** The compatibility matrix describes generation routes, not universal native installation. `install` prevalidates the complete selection and rejects routes whose persistent native mapping is not implemented:

- Copilot BYOK currently uses the isolated `task run` route. Although current Copilot documents `providers.json`, this installer does not guess its provider/model schema.
- Codex and OpenCode routes requiring extra cloud/environment variables currently require `task run` (or separately maintained ambient cloud setup). Pi can persist provider-scoped environment values in its native API-key credential; Claude stores them in `settings.json.env`. External ADC files, AWS profiles and identity refresh remain external dependencies.
- OpenCode V2 import is additive. The installer checks an existing database through native auth export and rejects a different credential or active selection for the same integration. Use OpenCode's auth commands to remove/select that credential before reinstalling; the installer does not mutate database tables directly. V2 model choice remains the CLI's own selection/`--model` behavior.
- Claude's native file bundle does not provision an OS keychain. Existing keychain credentials or higher-priority shell/project/managed settings can override installed files. API keys written to Claude settings follow its API billing behavior.

Installation resolves Pi API keys and Codex custom-provider bearer/header credentials into private native files so those routes work after the original environment variables are unset. Pi credential data is escaped to prevent it from being interpreted as a command or environment template. Installing does not mint subscriptions, validate model entitlements, or test provider inference.

All file plans and native command capabilities are checked before file writes. Unexpected failures roll back Credenvoy's own file replacements when they still match what it wrote. Native login/import may have external effects (including keychains) that cannot be rolled back as one transaction. Backups remain beside their targets for recovery. Keep them private and remove them after verifying the new setup.

Native references: [Copilot token login](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference), [Copilot configuration](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference), [Pi providers](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/providers.md), [Codex configuration](https://developers.openai.com/codex/config-reference/), [examined OpenCode V2 importer](https://github.com/anomalyco/opencode/blob/b6cbbd7646ec7a05945db3e7f499c9ae5af0b3d1/packages/cli/src/commands/handlers/auth/import.ts).

### Try isolated generation without an API key

```sh
# Creates a local-model profile; refuses to overwrite an existing profiles.json.
task init
```

Edit `profiles.json` and replace `your-local-model` with the model ID served by your local endpoint. Then:

```sh
export LOCAL_BASE_URL=http://127.0.0.1:11434/v1

task plan
task generate
```

This creates a private `generated/pi-local/` directory. **Generation does not contact the endpoint or launch an agent.** With Pi installed and your local service running:

```sh
AGENTCFG_PROFILE=pi-local task run
```

The examples above use a POSIX shell. In PowerShell, set variables with `$env:LOCAL_BASE_URL = 'http://127.0.0.1:11434/v1'` and `$env:AGENTCFG_PROFILE = 'pi-local'`, then run the same tasks.

### Use an OpenAI API key

Print a different starter profile:

```sh
AGENTCFG_EXAMPLE=openai task example
```

Save that JSON as `profiles.json`, or select another file with `AGENTCFG_SPEC`. Supply `OPENAI_API_KEY` through your usual secret-management mechanism and set `OPENAI_MODEL` to the model ID you want to use.

```sh
task plan
task generate
AGENTCFG_PROFILE=codex-openai task run
```

Use a fresh output directory for each reviewed generation:

```sh
AGENTCFG_OUTPUT=generated/review-2 task generate
AGENTCFG_OUTPUT=generated/review-2 AGENTCFG_PROFILE=codex-openai task run
```

Credenvoy refuses to overwrite an existing generation directory. Normal sessions should reuse their generated directory so the CLI can retain refreshed credentials.

## A profile is deliberately small

```json
{
  "profiles": [
    {
      "name": "pi-groq",
      "agent": "pi",
      "provider": "groq",
      "auth": "api_key",
      "model_env": "GROQ_MODEL"
    },
    {
      "name": "codex-chatgpt",
      "agent": "codex",
      "provider": "openai-chatgpt",
      "auth": "subscription",
      "model_env": "CHATGPT_MODEL"
    }
  ]
}
```

The first profile resolves `GROQ_API_KEY`. The second requires a complete native subscription credential bundle in `CODEX_AUTH_JSON`; it does not select API billing.

Model IDs are provider-specific. Credenvoy requires your choice rather than guessing a current model or assuming every catalog entry supports coding-agent tools.

## Commands

| Command | What it does |
|---|---|
| `task init` | Create a starter `profiles.json`, without overwriting it |
| `task example` | Print an example selected by `AGENTCFG_EXAMPLE` |
| `task examples` | Print the generated synthetic credential walkthrough |
| `task examples:write` | Regenerate `EXAMPLES.md` |
| `task examples:check` | Check the walkthrough for drift |
| `task matrix` | Print every provider/auth/CLI route as JSON |
| `task matrix:markdown` | Print the same matrix as a readable Markdown table |
| `task validate` | Validate selected inputs without writing configuration |
| `task plan` | Validate and list the native files that will be generated |
| `task install:plan` | Preview native installation destinations without changing CLI state |
| `task install` | Install selected credentials and configuration in normal CLI locations |
| `task generate` | Generate all selected profiles into a new private directory |
| `task run` | Launch `AGENTCFG_PROFILE` with its generated configuration |
| `task lint` | Check embedded runtime syntax and catalog invariants |
| `task test` | Run the embedded tests and every compatibility-matrix cell |
| `task smoke` | Generate a synthetic local profile in temporary storage |
| `task check` | Run lint, tests, and the offline smoke check |
| `task export` | Optionally export a standalone standard-library Python engine |

Available examples: `local`, `openai`, `suite`, `gateway`, `codex-to-opencode-legacy`.

### Selection and overrides

| Environment variable | Purpose | Default |
|---|---|---|
| `AGENTCFG_SPEC` | Profile JSON file | `profiles.json` |
| `AGENT_CONFIG_SPEC` | Complete profile document as JSON; overrides the file | Unset |
| `AGENTCFG_OUTPUT` | New generation directory | `generated` |
| `AGENTCFG_PROFILE` | Generated profile to launch | Required for `run` |
| `AGENTCFG_EXAMPLE` | Starter example to print or initialize | `local` |
| `AGENTCFG_EXPORT` | Optional standalone export filename | `credenvoy.py` |

Profiles may specify `model` directly, or `model_env`. Without either, the compiler expects `AGENTCFG_<AGENT>_<PROVIDER>_MODEL`, with names uppercased and hyphens replaced by underscores.

API-key source precedence is explicit `key_env`, then `AGENTCFG_<AGENT>_<PROVIDER>_API_KEY`, then the provider's conventional variables. Conflicting nonempty aliases fail. Secrets are read in Python; they are never inserted into shell command templates.

`.env` files are **not loaded automatically**. Export values yourself or use your secret manager's environment-injection command.

## Generated examples

The paths in [EXAMPLES.md](EXAMPLES.md) describe isolated `generate`/`run` output; `task install` writes directly to the native locations listed above. See that walkthrough for synthetic source credentials, selected profiles, generated files, and runtime destinations—including Codex subscription auth, legacy OpenCode V1/V2 conversion, native V2 token sharing, Pi API-key references, and Copilot bootstrap.

```sh
task examples          # print the walkthrough
task examples:write    # refresh EXAMPLES.md
task examples:check    # detect documentation drift
```

The walkthrough is rendered with inherited environment values blocked. It does not read live login caches or install credentials. V1’s normal `~/.local/share/opencode/auth.json` path is shown alongside Credenvoy’s isolated state; V2 examples use a credential import and SQLite.

## Subscription access

Complete target-native bundles preserve the login method, account metadata, refresh authority, and provider-specific fields.

| Service | CLI | Default source |
|---|---|---|
| ChatGPT subscription | Codex | `CODEX_AUTH_JSON` |
| ChatGPT subscription | OpenCode V1 | `OPENCODE_V1_OPENAI_OAUTH_JSON` |
| ChatGPT subscription | OpenCode V2 | `OPENCODE_OPENAI_AUTH_EXPORT_JSON` |
| ChatGPT subscription | Pi | `PI_OPENAI_CODEX_OAUTH_JSON` |
| Claude subscription | Claude Code | `CLAUDE_AUTH_JSON` |
| Copilot upstream OAuth | OpenCode, Pi, Copilot CLI | `GITHUB_COPILOT_OAUTH_TOKEN` |
| Supported GitHub token | Pi, Copilot CLI | `COPILOT_GITHUB_TOKEN` |

Use `credential_env` to select another source. For Claude Code, explicitly selecting `CLAUDE_CODE_OAUTH_TOKEN` uses its native runtime token.

OpenCode's current ChatGPT token-sharing credentials require their original method, client ID, and granted token-sharing scope. Renaming Codex fields cannot create those properties. The opt-in legacy converter separately requires known matching issuer/client provenance and an explicit access-token expiry in Unix milliseconds. Print its example with:

```sh
AGENTCFG_EXAMPLE=codex-to-opencode-legacy task example
```

If provenance is absent from `tokens.oauth_provenance`, it must be deliberately supplied in the profile's `oauth_provenance` object. Legacy conversion has structure tests; live cross-client refresh was not tested.

## Compatibility

The complete matrix is generated from the same code that selects adapters:

```sh
task matrix:markdown
```

| Route | Meaning |
|---|---|
| `native-api`, `native-azure`, `native-cloud` | Generate the selected integration's credentials and routing configuration |
| `native-bundle`, `native-token` | Preserve native subscription state or use a target-specific token route |
| `compatible-endpoint` | Configure an endpoint speaking a protocol the receiving CLI supports |
| `gateway-required` | Require an explicit compatible service and its own credential source |
| `unsupported-native`, `unsupported-auth` | Reject the selected combination |

The current registry contains **189 native/compatible configuration routes, 86 gateway-required combinations, and 13 unsupported combinations**. These are generation classifications, not 288 verified live integrations. A provider may offer another API that is not wired into the default registry.

- Claude Code's custom endpoint route requires a suitable Messages API.
- Codex custom providers require Responses compatibility in the examined contract.
- Copilot BYOK requires a supported protocol plus model streaming and tool support.
- Cloud credentials can depend on AWS profiles/roles, Google ADC, or Azure identity outside the generated directory.
- Subscription entitlement, billing, model availability, and release-level interoperability require live verification.

### A gateway is an explicit choice

```json
{
  "profiles": [{
    "name": "codex-groq-gateway",
    "agent": "codex",
    "provider": "groq",
    "auth": "api_key",
    "model_env": "GROQ_GATEWAY_MODEL",
    "endpoint": {
      "protocol": "openai-responses",
      "base_url_env": "GROQ_GATEWAY_URL",
      "key_env": "GROQ_GATEWAY_KEY"
    }
  }]
}
```

Credenvoy configures this route; it does not implement the gateway. Its credential source is required explicitly. For an unauthenticated endpoint, use `keyless: true` instead of `key_env`. Localhost may use HTTP; remote endpoints require HTTPS.

## What happens when you run a profile?

1. Resolve the selected runtime environment and remove known inherited model credentials/provider selectors.
2. Point the CLI at the profile's generated state directory.
3. Bootstrap credentials where needed, then preserve subsequent refreshes.
4. Launch an argument array directly, without a shell.

OpenCode V1 and V2 are separate adapters. V2 uses an isolated database and imports credentials once through a supported standalone command. If the installed CLI cannot import in standalone mode, Credenvoy refuses to target a shared background server.

Files are created with Unix mode `0600`, directories with `0700`. The launcher checks OpenCode's major version and rejects old Pi versions with different env-reference semantics. Windows ACL hardening and other release-level compatibility checks remain limitations; Windows CI tests configuration behavior, not live credential isolation.

CLI version detection accepts bare versions and a leading `v`, including OpenCode V2's `opencode v2.0.28` output. This check is shared by isolated launches and native installation; a different OpenCode major version is still rejected.

Credenvoy does not synchronize rotating OAuth state between machines, log into cloud providers, create API keys, or change your default home-directory agent configuration. Project-local settings and external cloud identity sources can still influence the agent itself.

## Small repository, explicit responsibilities

```text
Taskfile.yml             Public commands
README.md                Usage and compatibility
AGENTS.md                Development guide for humans and agents
LICENSE / NOTICE         License and upstream attribution
SECURITY.md              Credential-handling and reporting guidance
tasks/
  runtime.yml            Compiler, serializers, launcher, and dispatcher
  catalog.yml            Provider registry, env names, and examples
  tests.yml              Synthetic unit and end-to-end tests
.github/
  workflows/             Cross-platform CI and source-bundle checks
  dependabot.yml         Pinned action update proposals
```

Copy `Taskfile.yml` and `tasks/` to another directory and the same commands work. An optional `task export` creates a standalone `credenvoy.py`; use `python credenvoy.py --help` to see its commands. Neither approach needs an installed Credenvoy package or a runtime dependency download.

## Development

Read [AGENTS.md](AGENTS.md), then run:

```sh
task check
```

The embedded tests cover all 288 matrix cells, subscription preservation, legacy conversion, serialization, aliases, cloud credential groups, no-overwrite behavior, isolated runtime selection, bootstrap/import capability checks, native installation, merging, backups, rollback, JSONC/TOML preservation, and a real plan/generate subprocess round trip using synthetic credentials.

GitHub CI is configured for Python 3.11–3.14 on Linux, with additional macOS and Windows jobs. It copies only the Taskfiles into a temporary directory before running checks, so accidental dependencies on other repository files fail early. CI requires no provider credentials and makes no live inference or OAuth requests.

## Sources and scope

Provider hints originate from [models.dev](https://github.com/anomalyco/models.dev) at commit `c0c9177c6db1da0a17cca457bf2e80a8e5b911d7`. The embedded registry retains provider source links. Authentication contracts were researched against [Claude Code](https://code.claude.com/docs/en/authentication), [OpenCode](https://opencode.ai/v2/docs/providers/), [Pi](https://github.com/earendil-works/pi), [Codex](https://learn.chatgpt.com/docs/auth), and [Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/authenticate-copilot-cli).

The suite is a configuration generator with synthetic validation. Live provider behavior may change independently of this repository. Contribute a documented route and its tests when adding or updating an integration.

## License

[MIT](LICENSE). Upstream provider-data attribution is recorded in [NOTICE](NOTICE).
