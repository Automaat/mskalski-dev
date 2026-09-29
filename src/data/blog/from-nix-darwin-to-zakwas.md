---
title: From nix-darwin to zakwas
description: Why I replaced nix-darwin with a small Go CLI for my Mac setup, and what it takes to turn a personal script into a tool other people can use.
pubDatetime: 2026-09-29T00:00:00Z
tags:
  - macos
  - dotfiles
  - go
  - open-source
draft: true
---

For a couple of years my Mac was configured by nix-darwin and home-manager. It worked, and I liked the core idea: the machine is whatever the repo says, and dotfiles are read-only, so the only way to change them is through the repo. What I didn't like was everything around that idea. Every change went through a language I only half knew, error messages pointed into the middle of nixpkgs, and a routine update could turn into an evening.

So I asked a simpler question: which parts of Nix was I actually using?

- dotfiles installed as read-only copies
- a list of GUI apps
- pinned CLI tool versions that Renovate bumps
- a handful of macOS preferences
- a few one-off setup steps (SSH key, Touch ID for sudo)

Homebrew already handles apps with `brew bundle`. mise already handles pinned CLI tools, and Renovate understands its config. What was missing was the glue: something that reads one file, shows me what would change, and makes it so. That glue became **zakwas**.

## What it does

You describe the machine in `zakwas.yaml` and keep it in a git repo:

```yaml
protect:
  immutable: true
files:
  - { src: dotfiles/zshrc, dst: ~/.zshrc }
brew:
  file: Brewfile
  cleanup: zap
mise:
  config: dotfiles/mise/config.toml
defaults:
  - { domain: com.apple.dock, key: autohide, value: true }
```

`zakwas plan` shows what would change, `zakwas apply` does it, and `zakwas check` exits 2 when the machine has drifted, which is how a daily launchd job tells me that something changed behind my back.

```
$ zakwas plan
files
  ~ ~/.zshenv                         content
mise
  ~ ruff          0.16.2 → 0.16.9
  ▶ mise install
✓ up to date: system, links, templates, brew, commands, defaults

Plan: 0 to add, 2 to change, 0 to remove, 1 to run.
```

## Keeping what I liked about Nix

The part of Nix I wanted to keep was the read-only dotfiles. zakwas copies each file into place with the write bits removed and sets the macOS `uchg` flag, so even `:w!` in vim fails. It also records a hash of everything it writes, which lets it tell three situations apart:

- the repo changed: overwrite
- someone edited the installed copy anyway: back it up to `file.zakwas-bak`, then restore
- the file was dropped from the config: delete it, or back it up first if it was edited

Nothing it didn't write gets deleted or silently overwritten.

## A new Mac is one command

```bash
curl -fsSL https://raw.githubusercontent.com/Automaat/zakwas/main/install.sh |
  bash -s -- --repo https://github.com/you/dotfiles.git
```

The installer only installs the Xcode Command Line Tools and zakwas itself, after checking the release checksum. zakwas then notices that Homebrew and mise are missing and plans their installation as the first steps: Homebrew from its official installer, mise from a pinned release checked against its published SHA256 sums. CI runs this on a clean macOS runner for every change.

## From personal script to product

The first version lived inside my dotfiles repo and was called `eac` (environment as code). Moving it into its own repo, [Automaat/zakwas](https://github.com/Automaat/zakwas), meant removing every assumption about me: no personal paths or taps in the code, an example config that CI actually applies, releases with checksums and build provenance, a JSON Schema so editors can autocomplete the config, `zakwas init` to build a starting config from the Mac you already have, and machine-readable output (`plan --json`, saved plans) for anyone who wants to wire it into CI.

My own config now lives in [environment-as-code](https://github.com/Automaat/environment-as-code) and is just YAML, dotfiles and a Brewfile.

## Why "zakwas"

_Zakwas_ is Polish for sourdough starter. You keep it, feed it now and then, and every loaf you bake from it comes out the same. That's what I want from a machine setup.

It's early (pre-1.0) and macOS only. If you try it, I'd like to hear what's missing: [open an issue](https://github.com/Automaat/zakwas/issues).
