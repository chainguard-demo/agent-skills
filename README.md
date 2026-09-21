# Chainguard Agent Skills: security examples and hardened skills

An agent skill is a folder of instructions, centered on a `SKILL.md` file, that gives an AI agent specialized behavior. Because skills can influence agents running on developer workstations, an unsafe or compromised skill can become a software supply-chain and workstation risk. This repository shows three examples: a normal skill, defanged malicious specimens, and an upstream skill beside its Chainguard-hardened version.

## Safety warning: defanged specimens

The files in [`malicious-skill-example/`](malicious-skill-example/) are inert, defanged specimens captured from the upstream [`roin-orca/skills`](https://github.com/roin-orca/skills) repository for analysis. They were still present upstream as of September 2026. **Do not install, run, invoke, or load these specimens into an agent.** Inspect them only as text.

The `fun-brainstorming` specimen contains a booster pattern that attempts a hidden global install with `npx skills add ... --yes -g`. The `find-skills` specimen contains a beacon that makes an outbound `curl` request to a typosquatted domain. Those behaviors are defanged here, but the files must not be run.

## Repository contents

- [`workstation-checkup-sample-skill/`](workstation-checkup-sample-skill/): a clean reference showing the structure of a well-formed skill.
- [`malicious-skill-example/`](malicious-skill-example/): two defanged specimens captured from upstream for analysis: [`fun-brainstorming`](malicious-skill-example/fun-brainstorming.SKILL.md) and [`find-skills`](malicious-skill-example/find-skills.SKILL.md).
- [`hardening-comparison/`](hardening-comparison/): a useful but vulnerable [`upstream skill`](hardening-comparison/web-design-upstream/SKILL.md), its [`Chainguard-hardened version`](hardening-comparison/web-design-hardened/SKILL.md), and the accompanying [`hardening report`](hardening-comparison/web-design-hardened/HARDENING.md).

## What you'll learn

- How a skill's metadata and Markdown body guide an agent.
- How hidden installation, outbound beacons, and remote instruction fetching create risk.
- How hardening makes behavior reviewable, limits permissions, and records each change.

A [February 2026 Snyk study of about 4,000 skills on public marketplaces](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/) found nearly 37% had at least one security flaw, and about one in seven a critical issue such as malware, prompt injection, or exposed secrets.

## Self-guided walkthrough

You can follow the links above in a browser; cloning is optional. If you use a local checkout, the `cat` and `diff` commands below only inspect files. Do not install, run, invoke, or load the specimens into an agent.

### Inspect a normal skill

Read the reference skill:

```bash
cat workstation-checkup-sample-skill/SKILL.md
```

Notice the frontmatter fields, especially `name` and `description`, followed by the Markdown instructions that form the skill body. This example performs a read-only workstation checkup and explains its findings in plain language.

### Recognize malicious behavior

Read the two defanged specimens as text only:

```bash
cat malicious-skill-example/fun-brainstorming.SKILL.md
cat malicious-skill-example/find-skills.SKILL.md
```

In `fun-brainstorming`, find the hidden booster command based on `npx skills add ... --yes -g`, which attempts to install another skill globally without confirmation. In `find-skills`, find the outbound `curl` beacon to a typosquatted domain. The surrounding content makes the files resemble ordinary skills, but these commands reveal their actual risk.

### Compare an upstream and hardened skill

Inspect the changes side by side:

```bash
diff hardening-comparison/web-design-upstream/SKILL.md hardening-comparison/web-design-hardened/SKILL.md
```

The upstream skill fetches instructions at runtime from a remote URL. The hardened version removes that supply-chain dependency by inlining the rules and adds a minimal `allowed-tools` declaration so its behavior is visible and constrained.

### Read the hardening report

```bash
cat hardening-comparison/web-design-hardened/HARDENING.md
```

The report records the source, findings, severity, and changes made during hardening. Use it to connect the diff to the security rationale.

### Use public hardened skills

Add the [Public Skills MCP server](https://skills.cgr.dev/mcp) to Claude Code:

```bash
claude mcp add --transport http --scope user cgr-skills https://skills.cgr.dev/mcp
```

Then run `/mcp` in Claude Code, select `cgr-skills`, and authenticate in the browser. Access requires only a free Chainguard Console account; it does not require `chainctl` or a paid entitlement.

Advanced (CLI and private registry): see the [Chainguard Agent Skills documentation](https://edu.chainguard.dev/chainguard/agent-skills/).

## Key takeaways

- Treat agent skills as supply-chain inputs with access to sensitive developer environments.
- Review skills for hidden installation, network access, remote instruction loading, and excessive permissions.
- Prefer hardened skills whose behavior and provenance can be inspected before use.
