---
name: workstation-checkup-sample-skill
description: Use when a non-technical user asks for a "security checkup", "audit my computer", "is my laptop set up safely", or otherwise expresses general worry about how their machine is configured. Plain-English security review for a personal Linux or macOS laptop. Produces a checkup report for someone without a technical background. Read-only.
---

# Workstation Checkup (Plain English)

A friendly read-only security checkup for a personal laptop. The output is written for someone who is not a developer or sysadmin: no jargon, no scary log dumps, plain English with concrete next steps where things need attention.

## Worked example

> User: "Can you give my laptop a once-over? I just got a new one and I want to make sure I haven't done anything dumb."

The skill runs through five plain-English questions in order. On a typical fresh consumer laptop it might find: macOS firewall is off (Should fix), nothing saved as plain text passwords (Looking good), software up to date (Looking good), one unrecognized auto-starting app turns out to be a printer utility (Worth a look), no sensitive-file changes recently (Looking good), one check needed the user's password and got skipped (instructions provided). Writes a short plain-English report to `./checkup.md`. Final reply to the user: "Your laptop looks healthy overall. The one thing I'd fix today is the firewall — happy to walk you through it if you want."

## Hard constraints

- **Read-only.** Don't install, change, restart, or update anything. The checkup is the deliverable.
- **Plain English in the report.** Don't paste raw command output. Translate every finding into one or two sentences a non-technical person can act on.
- **Three severity buckets**: "Looking good", "Worth a look", "Should fix". No three-letter tags.
- **Don't print secrets.** If you find an API key or token, name the file. Never the value.
- **No remediation in this turn.** If the user wants to fix things, that's a separate request. Offer at the end; don't act.
- **One file.** Default `./checkup.md`.

## When to use

The user is not a developer, or is a developer asking about a non-work personal machine, and uses casual phrasing. If they use technical vocabulary and want raw findings rather than plain English, this checkup isn't the right fit — say so and offer a more technical review instead.

## Procedure

### Step 0 — detect OS

`uname -s`. Linux or Darwin (macOS). Anything else: tell the user this checkup targets personal Linux and macOS laptops, and stop.

### Step 1 — Is anything on this computer accepting connections from outside?

Listening services and the firewall.

- Linux: `ss -tlnp`, `ss -ulnp`, `systemctl is-active ufw`.
- macOS: `lsof -iTCP -sTCP:LISTEN -P -n`, `defaults read /Library/Preferences/com.apple.alf globalstate`.

Translate:
- Things bound to `127.0.0.1` are only reachable from this computer. Note as fine.
- Things bound to `0.0.0.0` or `*` are reachable from any network the laptop joins. Worth a look — explain in plain words what each one is.
- Firewall on → good. Firewall off → "Should fix", with a one-sentence reason ("turning it on means programs on other computers can't reach yours unless you say so").

If Docker is installed, mention as a note that Docker can publish ports without going through the firewall — useful background in case they ever wonder why something's still reachable.

### Step 2 — Are any passwords or login keys saved as plain text?

Look in the obvious places:
- `~/.zshrc`, `~/.bashrc`, `~/.profile` for `export ...=` lines that look like API keys (long random-looking strings; JWT format; values starting with `sk-`, `lin_api_`, `xoxp-`, `xoxb-`, `ghp_`, `glpat-`, `AKIA`).
- `~/.docker/config.json` for an `auths` block.
- `~/.aws/credentials` for static keys (vs an SSO config, which is fine).
- `~/.netrc` if it exists.
- `~/.ssh/` for files named `id_*` or `*.pem`. For each: `ssh-keygen -y -P '' -f FILE` succeeding means the key has no passphrase.

Translate:
- "I found an API key for [SERVICE] saved as plain text in [FILE]. Anything that can read your home folder could use it. The fix is to move it into your computer's keyring — happy to walk you through it if you want."
- "Your SSH key at [FILE] has no passphrase. If your laptop is ever unlocked while someone else has it, they can use it."
- All clear: "Looking good — no passwords saved as plain text where any program could read them."

### Step 3 — Is your software up to date and from trusted sources?

- Linux: `apt list --upgradable`, `ls /etc/apt/sources.list.d/`, `snap list`.
- macOS: `brew outdated`, `brew tap`, `mas outdated` (if installed). Also `csrutil status`, `spctl --status`, `fdesetup status` for SIP / Gatekeeper / FileVault.

Translate:
- Pending **security** updates: "Should fix — there are X security updates waiting. Apply them when you can."
- Software sources: list publishers in plain words ("Apple, Homebrew, Microsoft, Google Chrome"). Anything unrecognized: "Worth a look — there's a software source called [X]. If you don't remember installing software from them, let me know."
- macOS specifically: Gatekeeper off → Should fix. SIP disabled → Should fix. FileVault off → Should fix (it's the disk encryption that protects a stolen laptop).

### Step 4 — Are unfamiliar programs running on a schedule or at startup?

- Linux: `crontab -l`, `systemctl list-timers --all`, `ls ~/.config/autostart/`.
- macOS: `crontab -l`, `launchctl list`, `ls ~/Library/LaunchAgents`, login items via `osascript`.

Translate. Standard system stuff (Ubuntu's apt-daily, Apple's `com.apple.*` daemons) gets a one-line "the usual system maintenance" mention. Anything else: name it and say what it does if you can identify it.

### Step 5 — Have any sensitive files changed recently in places they shouldn't?

`find ~/.ssh ~/.gnupg -mtime -30` and `find /etc -mtime -30 | head -30` (on macOS also `/private/etc`).

Translate:
- Activity in `~/.ssh` they don't remember: "Worth a look — your SSH key folder had files changed on [date]. Was that you?"
- `/etc` changes that are clearly from system updates: "the usual system updates — nothing handwritten."

### Step 6 — Compile the checkup

Write `./checkup.md`:

```
# Computer Checkup

Date: YYYY-MM-DD
Computer: <hostname>

I looked at five things on your computer to see if anything looked off.
Each item below is one of:

- "Looking good" - nothing to do
- "Worth a look" - probably fine, but worth knowing about
- "Should fix" - take care of this when you can

---

## 1. Is anything on your computer accepting connections from outside?
[plain explanation of what was found]

## 2. Are any passwords or keys saved as plain text?
...

## 3. Is your software up to date and from trusted sources?
...

## 4. Are unfamiliar programs running on a schedule?
...

## 5. Have any sensitive files changed recently?
...

---

## What to fix first
A short ranked list of "Should fix" items, with one or two sentences per item explaining what to do. Friend-not-sysadmin language. "Go to [website] and click the button that says X" beats "rotate the credential."

## What needs your help
A short list of things that needed your password and got skipped, as commands to paste into the terminal. Be honest about why we couldn't check them automatically: "These checks need your computer's password, which I can't enter. Run the command below in your terminal — it'll ask for your password and then check the rest."
```

### Step 7 — Closeout

End the reply with:

1. "I wrote your checkup to `<path>`. The short version: <2–3 sentence summary>."
2. The top one to three "Should fix" items as a list, each one sentence.
3. "If you want me to help fix any of these, just say which one." Nothing more.

Don't lecture. Don't fix things in the same turn. Don't list every "Looking good" item — that's filler.

## Adapting

If the user has Docker, AWS, GCloud, or other developer tooling installed, you can include it but explain what it is in one sentence the first time you mention it. If the user is clearly more technical than this skill assumes (jumps in with "what about ufw rules"), you can go into more depth, but keep the written report in plain English.

## Out of scope

- Windows.
- Anything that requires the user to know what a "process" or "port" is in advance — explain inline if you have to mention either one.
- Compliance frameworks. Wrong audience.
- Active investigation of a suspected compromise. Different skill, different stakes.
