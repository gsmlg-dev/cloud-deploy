# Cloud Deploy

GitHub Actions workflow for deploying NixOS configurations to Vultr cloud hosts.

## Architecture

```
GitHub Actions (ubuntu-latest)
        │
        ├── Clone: Gao-OS/nix-config (private)
        │
        └── Deploy via SSH + nixos-rebuild
                │
    ┌───────┬───┼───────┐
    ▼       ▼   ▼       ▼
vultr-01 vultr-02 vultr-03 vultr-04
```

## Target Hosts

| Host | Purpose |
|------|---------|
| vultr-01 | General purpose |
| vultr-02 | Proxy infrastructure (Squid) |
| vultr-03 | Web services (Caddy + Rails + YellowDog DNS) |
| vultr-04 | Web services (Caddy + DNS) |

## Setup

### 1. Generate SSH Key for Deployments

```bash
ssh-keygen -t ed25519 -C "cloud-deploy@github-actions" -f ~/.ssh/cloud-deploy -N ""
```

### 2. Add Public Key to Target Hosts

On each target host (vultr-01, vultr-02, vultr-03, vultr-04):

```bash
# Add to authorized_keys
echo "ssh-ed25519 AAAA... cloud-deploy@github-actions" >> ~/.ssh/authorized_keys
```

Or add to your NixOS configuration:

```nix
users.users.gao.openssh.authorizedKeys.keys = [
  "ssh-ed25519 AAAA... cloud-deploy@github-actions"
];
```

### 3. Create Deploy Key for nix-config Repository

```bash
ssh-keygen -t ed25519 -C "cloud-deploy-nix-config" -f ~/.ssh/nix-config-deploy -N ""
```

Add the public key to https://github.com/Gao-OS/nix-config/settings/keys as a deploy key (read-only).

### 4. Get SSH Known Hosts

```bash
ssh-keyscan vultr-01.gsmlg.net vultr-02.gsmlg.net vultr-03.gsmlg.net vultr-04.gsmlg.net
```

### 5. Configure GitHub Secrets

Go to: Repository Settings > Secrets and variables > Actions

| Secret | Value |
|--------|-------|
| `SSH_PRIVATE_KEY` | Contents of `~/.ssh/cloud-deploy` |
| `SSH_KNOWN_HOSTS` | Output from ssh-keyscan command above |
| `NIX_CONFIG_DEPLOY_KEY` | Contents of `~/.ssh/nix-config-deploy` |

### 6. Configure Target Hosts

Ensure each target host has these NixOS settings:

```nix
{
  # Allow gao user to receive unsigned store paths
  nix.settings.trusted-users = [ "root" "gao" ];

  # Passwordless sudo for nixos-rebuild
  security.sudo.extraRules = [{
    users = [ "gao" ];
    commands = [{
      command = "/run/current-system/sw/bin/nixos-rebuild";
      options = [ "NOPASSWD" ];
    }];
  }];
}
```

## Usage

### Via GitHub UI

1. Go to **Actions** tab
2. Select **Deploy NixOS** workflow
3. Click **Run workflow**
4. Choose target:
   - `vultr-01`, `vultr-02`, `vultr-03`, `vultr-04` - Deploy to single host
   - `all` - Deploy to all hosts in parallel
5. Optionally enable "Build on target host"
6. Click **Run workflow**

### Via GitHub CLI

```bash
# Deploy to a single host
gh workflow run deploy.yml -f target=vultr-02

# Deploy to all hosts
gh workflow run deploy.yml -f target=all

# Deploy with remote build (uses target's resources)
gh workflow run deploy.yml -f target=vultr-03 -f build_on_target=true

# Watch progress
gh run watch
```

## Build Strategies

### Local Build (Default)

Build on GitHub Actions runner, push closure to target.

- Doesn't consume target resources during build
- Larger data transfer

### Remote Build

Build directly on target host.

- Minimal data transfer
- Uses target's existing Nix store cache
- Enable with `build_on_target: true`

## Troubleshooting

### "lacks a signature by a trusted key"

Ensure `nix.settings.trusted-users` includes the deployment user on target hosts.

### SSH Connection Refused

Verify SSH access works:

```bash
ssh gao@vultr-02.gsmlg.net 'echo OK'
```

### Deployment Timeout

For slow builds, use remote build option or consider setting up a binary cache (Cachix).
