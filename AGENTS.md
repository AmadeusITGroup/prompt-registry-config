# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-file configuration repository. The only meaningful artifact is `hub-config.yml`, which defines the Community **Prompt Registry Hub** — a centralized catalog of AI prompt/skill bundles via the VS Code Prompt Registry extension.

## Validation

CI runs two checks on every push/PR touching YAML files:

```bash
# Lint YAML syntax
yamllint -c .config/yamllint .

# Validate against the hub-config JSON schema
check-jsonschema \
  --schemafile "https://raw.githubusercontent.com/AmadeusITGroup/prompt-registry/refs/heads/main/schemas/hub-config.schema.json" \
  hub-config.yml
```

To run locally, install with `pip install yamllint check-jsonschema`.

## hub-config.yml structure

The file has four top-level sections:

- **`version`** — Hub config format version (currently `1.0.0`; required)
- **`metadata`** — Hub `name`, `description`, `maintainer`, `updatedAt` timestamp (update this on every change)
- **`sources`** — External repos providing skill/prompt bundles; each has a unique `id` referenced by profiles
- **`profiles`** — Named collections of bundles surfaced in the extension's navigation tree via `path` arrays

### Source types

Each source declares a `type` that tells the extension how to resolve its bundles. Common fields on every source: a unique `id` (referenced by each bundle's `source`), `name`, `url`, `enabled` (boolean), `priority` (0–100 integer), and an optional `metadata` block.

| `type` | Usage | Bundle id prefix |
|---|---|---|
| `skills` | A repository directory of Agent Skills (`url` ends in the skills dir). **Both sources in the current config use this.** | `skills-<org>-<repo>-<skillId>` |
| `github` | Plain GitHub repo; bundles resolved by directory names in a collection file | `<org>-<repo>-<bundleId>` |
| `awesome-copilot` | Structured collections repo configured with a `collectionsPath` (requires a `config` block with `branch`, `collectionsPath`) | `<bundleId>` (no org/repo prefix) |

### Sources in this config

Both sources currently in `hub-config.yml` are `type: skills`. Verbatim example:

```yaml
- id: awesome-copilot-skills-official      # unique, referenced by each bundle's `source`
  name: Awesome Copilot Skills (Official)
  type: skills
  url: https://github.com/github/awesome-copilot/skills   # https://github.com/<org>/<repo>/<skillsDir>
  enabled: true                            # boolean, required
  priority: 10                             # 0–100 integer, required
  metadata:
    description: Official Awesome Copilot Skills from GitHub
    homepage: https://github.com/github/awesome-copilot/skills
```

The two real sources are:

| `id` | `url` (org / repo / dir) | `priority` |
|---|---|---|
| `awesome-copilot-skills-official` | `github` / `awesome-copilot` / `skills` | 10 |
| `anthropic-skills-official` | `anthropics` / `skills` / `skills` | 20 |

### Bundle ID conventions

The bundle `id` you reference in a profile is auto-derived by the extension from the source — the safest way to confirm one is to install the source in the extension and inspect the ids it exposes. The pattern depends on the source `type`:

**`skills` sources** — join the literal prefix `skills`, the GitHub org, the repo, and the skill's own id (`skills-<org>-<repo>-<skillId>`). The org/repo segments come from the source `url`, **not** from the source `id`. Real examples from `hub-config.yml`:
- `skills-github-awesome-copilot-webapp-testing` — `skills` + org `github` + repo `awesome-copilot` + skill `webapp-testing` (source `awesome-copilot-skills-official`)
- `skills-anthropics-skills-frontend-design` — `skills` + org `anthropics` + repo `skills` + skill `frontend-design` (source `anthropic-skills-official`)

**`github` sources** — `<org>-<repo>-<bundleId>`, where `bundleId` is defined in the collection file. Example: a `commit-message` bundle in `microsoft/awesome-copilot` is referenced as `microsoft-awesome-copilot-commit-message`. _(No `github` source exists in the current config; shown for completeness.)_

**`awesome-copilot` sources** — just the bundle's own identifier, no org/repo prefix (`<bundleId>`). Example: a bundle with id `upgrade-and-migration` is referenced as `upgrade-and-migration`. _(No `awesome-copilot` source exists in the current config; shown for completeness.)_

## Profiles — how they work

A profile is a named, themed collection of bundles. The extension renders each profile as an entry under the tree location given by its `path`. Here is the `Frontend Developer` profile, verbatim from `hub-config.yml`:

```yaml
- id: frontend-developer                   # unique across all profiles
  name: Frontend Developer                 # display name in the tree
  description: Comprehensive frontend development tools including web frameworks, testing, and security
  icon: "🌐"                               # emoji shown next to the profile
  path: ["Roles"]                          # tree location (see below)
  bundles:
  - id: skills-github-awesome-copilot-webapp-testing   # must match the source's derived id
    version: latest                        # pin a version or use `latest`
    source: awesome-copilot-skills-official              # must equal an existing source `id`
    required: true
  - id: skills-anthropics-skills-frontend-design
    version: latest
    source: anthropic-skills-official
    required: true
  # ...more bundles
```

Each profile field:

- **`id`** — unique profile identifier (kebab-case in practice, e.g. `frontend-developer`, `devops-engineer`)
- **`name`** — human-readable label shown in the tree
- **`description`** — one-line summary of what the profile is for
- **`icon`** — a single emoji
- **`path`** — string array placing the profile in the navigation tree (see below)
- **`bundles`** — list of bundle references; each has:
  - **`id`** — the source-derived bundle id (`skills-<org>-<repo>-<skillId>`)
  - **`version`** — `latest`, or a pinned version
  - **`source`** — must match a source `id` declared in `sources`
  - **`required`** — boolean

The same bundle can appear in multiple profiles — e.g. `skills-github-awesome-copilot-webapp-testing` is referenced by both the `frontend-developer` and `testing-specialist` profiles.

### Profile `path` field

`path` is a string array naming the tree location where the profile appears. In `hub-config.yml` every profile uses a **single top-level category**, grouped in the file by comment headers:

| Category | Example profiles |
|---|---|
| `["Roles"]` | `frontend-developer`, `devops-engineer`, `ai-engineer`, `mcp-developer`, `designer-creative` |
| `["Domains"]` | `testing-specialist`, `ai-specialist`, `document-content-specialist`, `database-specialist`, `planning-architect` |
| `["Languages"]` | `javascript-developer`, `csharp-developer` |
| `["Platforms"]` | `azure-specialist`, `microsoft365-specialist`, `power-platform-specialist` |
| `["Tools"]` | `sdk-developer`, `meta-explorer` |

The schema also permits deeper nesting (e.g. `["Roles", "Developer", "Frontend_Developer"]`) if you want sub-categories, but the current config keeps every profile one level deep.

## Making changes

- Sources must have unique `id` values; profile `id` values must also be unique across the file
- Every bundle's `source` field must match an existing source `id`, and its bundle `id` must follow the convention for that source's `type` (see [Bundle ID conventions](#bundle-id-conventions)) — for the `skills` sources in this config that means `skills-<org>-<repo>-<skillId>`
- Keep new profiles grouped under the right comment header (`# Roles`, `# Domains`, `# Languages`, `# Platforms`, `# Tools`) so the file stays organized
- Update `metadata.updatedAt` on every change
- The schema reference at the top of `hub-config.yml` enables in-editor validation in IDEs that support yaml-language-server
