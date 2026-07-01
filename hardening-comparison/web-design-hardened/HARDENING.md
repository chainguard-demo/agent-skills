# Hardening Report: github.com/antfu/skills/skills/web-design-guidelines

| Field | Value |
|---|---|
| Upstream SHA | `5e3122fbe3b151c3e940a01900a82272184a769e` |
| Hardened at | 2026-06-12T23:52:21Z |
| Files processed | 3 |
| .md files (clean after harden) | 3 |
| .md files (attempts exhausted) | 0 |
| Non-.md files (copied verbatim) | 0 |
| Scanner findings addressed | 0 (0 applied) |

## Markdown files

### `LICENSE.md`

- Status: **clean**
- Attempts used: 1
- Engine findings: none (clean on first engine pass)

### `SKILL.md`

- Status: **clean**
- Attempts used: 3
- Engine findings + fixes applied:

  | Attempt | Rule | Severity | Finding |
  |---|---|---|---|
  | 1 | `no-obfuscated-commands` | high | The skill fetches its primary logic, including rules and output formatting instructions, from an external URL at runtime. This is a multi-stage stager pattern, where a payload is downloaded from 'https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md' and used to define the skill's behavior. This obscures the full execution logic from static analysis and introduces a significant security risk, as the external content can be modified to perform malicious actions. |
  | 2 | `minimal-permissions` | high | The skill definition does not specify an `allowed-tools` list. It only describes its purpose and functionality. Therefore, it's impossible to check for unnecessary tools. |

### `SYNC.md`

- Status: **clean**
- Attempts used: 1
- Engine findings: none (clean on first engine pass)

## Verbatim files

_None._
