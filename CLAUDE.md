# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

NixOS cloud deployment orchestration via GitHub Actions. Deploys NixOS configurations from the private `Gao-OS/nix-config` repository to Vultr cloud hosts using `nixos-rebuild` with Nix flakes.

## Architecture

GitHub Actions runner (ubuntu-latest) clones `Gao-OS/nix-config`, then deploys via SSH + `nixos-rebuild` to three Vultr hosts (`vultr-02`, `vultr-03`, `vultr-04`) under the `gsmlg.net` domain. Deployments use flake outputs (e.g., `.#vultr-02`). Matrix strategy enables parallel multi-host deployment.

Two build modes:
- **Local build** (default): builds on the GitHub Actions runner, pushes closure to target
- **Remote build** (`build_on_target: true`): builds directly on the target host

## Repository Structure

This is a minimal infrastructure-as-code repo with only GitHub Actions workflows:
- `.github/workflows/deploy.yml` — main deployment workflow (manual trigger via `workflow_dispatch`)
- `.github/workflows/claude.yml` — Claude Code integration for issue/PR interactions

## Deployment Commands

```bash
# Deploy to a single host
gh workflow run deploy.yml -f target=vultr-02

# Deploy to all hosts
gh workflow run deploy.yml -f target=all

# Deploy with remote build
gh workflow run deploy.yml -f target=vultr-03 -f build_on_target=true

# Watch deployment progress
gh run watch
```

## Key Configuration

- **SSH user**: `gao`
- **Domain**: `gsmlg.net`
- **Hosts**: `vultr-02` (Squid proxy), `vultr-03` (Caddy + Rails + DNS), `vultr-04` (Caddy + DNS)
- **Required secrets**: `SSH_PRIVATE_KEY`, `SSH_KNOWN_HOSTS`, `NIX_CONFIG_DEPLOY_KEY`
- **Nix config source**: `git@github.com:Gao-OS/nix-config.git`
