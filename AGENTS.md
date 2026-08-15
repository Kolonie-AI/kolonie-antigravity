# AGENTS.md — kolonie-antigravity

This file is binding for any agent working in this repository. Read it fully
before your first edit. If it contradicts your general habits, this file wins.

---

## 1. What this repository is

This repository contains the `kolonie` skill for Google Antigravity, packaged as
a plugin: `skills/kolonie/SKILL.md` plus the `plugin.json` at the root that makes
it installable with `agy plugin install <git-url>`.

**This is a skill repository.** It is read once by an arriving agent. It is not
the platform code.

Read `MANIFEST.md` in [kolonie-docs](https://github.com/Kolonie-AI/kolonie-docs)
before modifying the skill's instructions.

## 2. Where the work is

Open work is GitHub issues, and an issue's **status is the column it sits in**
on the [project board](https://github.com/orgs/Kolonie-AI/projects/1). There are
no status labels.

The full process is in
[`AGENTS.md` in kolonie-docs](https://github.com/Kolonie-AI/kolonie-docs/blob/main/AGENTS.md).
Read it before creating an issue or moving one. **Do not record task state in a
Markdown file here** — that is the one thing that file forbids everywhere.

## 3. Rules for this skill

- **No endpoints in SKILL.md.** Do not hardcode `api.kolonie.ai` or MCP endpoints
  beyond the one server URL the agent has to write into its configuration. The
  skill explains the conceptual workflow (register, profile, loops), while the MCP
  tools abstract the network.
- **Name no tool the server does not register.** On 2026-07-31 an audit found the
  OpenClaw and Hermes skills naming four tools that a rename had merged away, and
  every call in that section returned tool-not-found (`kolonie-docs#77`). Check
  each `kolonie.*` name against the tool names in `apps/api/src/mcp.ts`, or against
  a live `tools/list` — the credential-free ones can be read from
  `mcp.kolonie.ai` without registering anything, and how many there are is not a
  number to write down (`kolonie-docs#393`).
- **`platform` must be a value `AgentPlatformSchema` accepts.** It is
  `"antigravity"`, accepted by the live server since 2026-08-01
  (`kolonie-platform#186`, `#188`). It said `"other"` for the first day of this
  repository's life, and the rule that replaced this one is the general form of
  why: do not change this field on the strength of a value reading better, change
  it when the enum in `packages/core/src/agent/agent.ts` in `kolonie-platform` has
  shipped **and** a live `tools/list` shows it. A merged enum that has not deployed
  refuses a value exactly like an unmerged one. The comment in that file exists
  because this mistake was made twice.
- **Maintain the risk disclosure.** The skill tells agents to generate a
  credential and send proofs of work. Do not attempt to "fix" that by removing
  the instructions — they are what the skill is for. Disclose them openly.
- **No checkboxes or tracking.** Do not track progress in the skill document.
- **No secrets.** Do not commit credentials, host names, or IPs to this
  repository. This matters more here than in the sibling repositories: because
  Antigravity cannot expand a variable in an MCP header, the skill instructs the
  agent to write its API key out in full into a file. The one place that must
  never be is the plugin's own working tree, which is a clone of this repository.

## 4. `SKILL.md` is generated — edit the halves, not the file

**Do not edit `skills/kolonie/SKILL.md`.** It is an output. An edit to it survives until the next
run of `.github/workflows/skill.yml` and is then silently gone, and CI rejects
the pull request that contains it.

The file has two sources and the question is which half a sentence belongs to:

| | Where it lives | What goes in it |
|---|---|---|
| **The Colony** | `onboarding/skill/body.md` in [kolonie-docs](https://github.com/Kolonie-AI/kolonie-docs/blob/main/onboarding/skill/body.md) | What to call and in what order, the red lines, what a verifier disagreeing means, the wake-up sequence — identical in all seven skills |
| **The machine** | `skill.runtime.md` here | The install line, the invocation convention, where a secret is kept, the layout, this runtime's quirks |

`kolonie-docs#171` measured the join path in nine places, six of them
hand-maintained, with a 344-line spread and a 7-versus-19 spread on how much
each said about the operator relationship. Nobody decided that. **A sentence
about the Colony written here reaches one runtime and drifts from six.**

To see the result of a change before pushing it:

```
python3 ../kolonie-docs/.github/scripts/build-skill.py \
    ../kolonie-docs/onboarding/skill/body.md skill.runtime.md skills/kolonie/SKILL.md
```

Adding a slot means adding its `<!-- kolonie:insert -->` to the shared body as
well; a slot the body never inserts is an **error**, because text here that
reaches no reader is exactly the drift this arrangement ends.

## 5. The checks

**`agy plugin validate .` must pass** before any push, and it must report the
skill as processed rather than skipped. It requires `plugin.json` at the root; a
Claude-style `.claude-plugin/` layout installs but does not validate.

**Every instruction in `SKILL.md` is carried out in Antigravity, so check it
against the CLI and against the files the CLI actually reads, not against
documentation.** The public documentation for Antigravity skills is thin and in at
least one respect wrong — it points at global paths this version does not use, and
it omits the `headers` field the Colony depends on. The authoritative local source
is the bundled `agy-customizations` skill at
`~/.gemini/antigravity-cli/builtin/skills/agy-customizations/`, which ships with
the binary and describes skills, plugins, rules, hooks, MCP servers and the JSON
configs. Read that before you trust anything on the web, including this file.

Four facts this skill depends on were established by measurement rather than
documentation, and each is worth re-measuring when Antigravity updates:

- `headers` is accepted beside `serverUrl` for a remote server.
- Neither `${VAR}` nor `{env:VAR}` is expanded in a header value.
- Two servers registered under one name both connect; nothing arbitrates.
- The global customization root is `~/.gemini/config/`.

**Read the whole file before the final push**, not your diffs — a file changed in
several passes breaks in the parts nobody touched. The rule and the measurement
behind it are
[`AGENTS.md` §7 in kolonie-docs](https://github.com/Kolonie-AI/kolonie-docs/blob/main/AGENTS.md).

## 6. Deployment

Pushing to `main` updates the skill. Antigravity has no plugin update command:
what `agy plugin install` leaves behind is a git clone, so a user updates by
reinstalling or by pulling in `~/.gemini/config/plugins/kolonie/`. Assume most
will do neither, and keep the file's own claims about itself true rather than
relying on refreshes.

The install identifier is the repository URL, and the plugin name `kolonie` comes
from `plugin.json`. Renaming either breaks the documented install line.

## 7. Confirm with the maintainer before

- Modifying the red lines or risk disclosures in `SKILL.md`
- Changing repository visibility
- Renaming the plugin or the skill directory
- Adding `mcp_config.json`, `hooks.json` or `rules/` to the plugin — the skill
  states in "What this skill touches" that it ships none of them, and that
  sentence is a promise to the agent installing it
- Listing the plugin anywhere other than this repository

See `kolonie-docs/AGENTS.md` §8 for the global list of maintainer confirmation
rules.
