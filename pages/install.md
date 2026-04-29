---
layout: page
title: Install
eyebrow: Getting Workwarrior
subtitle: macOS and Linux. Bash or zsh. Python 3 for the browser UI. That's it.
permalink: /install
---

<div class="install-steps">

<div class="install-step">
<div class="step-body">

### Clone the repo

```bash
git clone https://github.com/babbworks/ww ~/ww
cd ~/ww
```

The repo is your installation. There's no package manager step — workwarrior lives in `~/ww` (or wherever you clone it).

</div>
</div>

<div class="install-step">
<div class="step-body">

### Run the installer

```bash
./install.sh
```

The installer presents a version card for each dependency: TaskWarrior, TimeWarrior, JRNL, Hledger, Bugwarrior, and the optional tools. For each one it shows installed version, latest available, and the minimum required.

On macOS it can install missing tools via Homebrew automatically. On Linux it shows the right command for your distro (apt, dnf, pacman).

</div>
</div>

<div class="install-step">
<div class="step-body">

### Source your shell config

```bash
source ~/.bashrc   # or source ~/.zshrc
```

The installer adds `ww-init.sh` to your shell config. This injects the `ww` command, shell functions (`task`, `timew`, `j`, `l`, `i`, `q`), and `p-<name>` aliases for each profile.

</div>
</div>

<div class="install-step">
<div class="step-body">

### Check dependencies

```bash
ww deps check
```

Shows the status of every dependency with version info. Install anything missing:

```bash
ww deps install
```

</div>
</div>

<div class="install-step">
<div class="step-body">

### Create your first profile

```bash
ww profile create work
p-work
```

You now have an isolated workspace at `~/ww/profiles/work/` with its own TaskWarrior database, TimeWarrior instance, journal directory, and ledger files. Every tool command (`task`, `timew`, `j`, `l`) operates within this profile.

</div>
</div>

<div class="install-step">
<div class="step-body">

### Launch the browser UI (optional)

```bash
ww browser
```

Opens `http://localhost:7777` with 15+ panels. No npm. No cloud. Python 3 stdlib only.

</div>
</div>

</div>

---

## Optional Extras

```bash
ww tui install       # taskwarrior-tui — full-screen terminal UI
ww mcp install       # MCP server — AI agent access to TaskWarrior data
```

---

## Platform Notes

| Platform | Shell | Status |
|----------|-------|--------|
| macOS (arm64, x86_64) | bash, zsh | Fully supported |
| Ubuntu/Debian | bash, zsh | Fully supported |
| Fedora/RHEL | bash, zsh | Supported |
| Arch Linux | bash | Supported |
| Windows (WSL2) | bash | Tested, supported |

---

## Updating

Workwarrior is a git repo. Update by pulling:

```bash
cd ~/ww
git pull
```

Profile data lives in `profiles/` which is gitignored — it never moves during an update.
