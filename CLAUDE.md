# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains community-contributed kits for Docker Sandboxes. Each top-level directory represents a kit that extends sandbox agents with additional capabilities. Kits are organized by kind:
- **Mixins**: Extend existing agents (e.g., mixins/brew, mixins/code-server)
- **Agents**: Standalone agent kits (e.g., amp, nanobot, nanoclaw, openclaw, pi, trivy)
- **Shared packages**: spec/ (kit artifact format) and tck/ (technology compatibility kit)

## Development Workflow

### Common Commands

**Kit Validation**
```bash
# Validate a specific kit's spec.yaml
sbx kit validate ./kit-name/

# Validate all kits (used in CI)
sbx kit validate ./...
```

**Local Testing**
```bash
# Run TCK tests for a specific kit
cd kit-name/
go test -v -count=1 -timeout 10m ./...

# Run tests for all kits
go test -v -count=1 -timeout 10m ./...
```

**Running Kits in Sandbox**
```bash
# Using local directory
sbx run --kit ./kit-name/ <agent>

# Using git reference (recommended for production)
sbx run --kit "git+https://github.com/docker/sbx-kits-contrib.git#ref=main&dir=kit-name" <agent>

# Pin to specific commit for reproducibility
sbx run --kit "git+https://github.com/docker/sbx-kits-contrib.git#ref=abc123&dir=kit-name" <agent>
```

**Kit Development**
1. Create directory: `mkdir kit-name`
2. Add `spec.yaml` and optional `files/` directory
3. Write TCK test: `kit-name_tck_test.go`
4. Validate locally before PR

### Code Structure

**Kit Format (`spec.yaml`)**
- `schemaVersion`: Version of kit schema
- `kind`: Either "mixin" or "agent"
- `name`: Kit identifier (lowercase, alphanumeric + hyphens)
- `displayName`: Human-readable name
- `description`: Brief description of kit functionality
- `network`: Allowed/denied domains for outbound connections
- `environment`: Variables to set in container
- `commands`: Install and startup commands
- `credentials`: Authentication requirements (for agent kits)
- `memory`: Persistent storage requirements
- `settings`: Configuration overrides
- `oauth`: OAuth provider configurations

**TCK Tests**
Each kit must include a `_tck_test.go` file that:
- Imports `github.com/docker/sbx-kits-contrib/tck`
- Creates a test suite from the kit directory
- Runs `suite.RunAll(t)` to validate:
  - Spec validation
  - Network policy
  - Credential policy
  - Commands
  - Environment variables
  - Container files
  - Security features (tmpfs mounts)

**Shared Packages**
- `spec/`: Go library for parsing/validating kit artifacts
- `tck/`: Test framework that validates kits against real containers

## Contribution Guidelines

### Commit Requirements
Every commit must have:
1. **DCO sign-off**: `Signed-off-by:` trailer (add with `git commit -s`)
2. **Cryptographic signature**: GPG or SSH signature (produces GitHub "Verified" badge)

**Configure automatic signing:**
```bash
# GPG signing (recommended)
git config --global commit.gpgsign true
# Then only need -s for commits
git commit -s -m "feat(kit-name): description"

# SSH signing (alternative)
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

### Pull Process
1. Read CONTRIBUTING.md before starting
2. Use existing kits as templates (mixins/code-server/ for mixins, amp/ for agents)
3. Every kit needs a README.md with:
   - Title and description
   - Usage instructions (`sbx run` examples)
   - Explanation of non-obvious design decisions
   - Cleanup instructions if kit creates host state
4. Run local validation before opening PR:
   ```bash
   sbx kit validate ./my-kit/
   cd my-kit && go test -v -count=1 -timeout 10m ./...
   sbx run --kit ./my-kit/ <agent>
   ```
5. Follow conventional commits:
   - New kit: `Add <kit-name> kit`
   - Fixes: `chore(<kit>): …`, `fix(tck): …`, `feat(spec): …`

### Kit Types
**Mixins**: Extend existing agents by adding capabilities to base templates (shell, claude, etc.)
- Use `extends` field in spec.yaml to specify parent agent
- Files in `files/home/` or `files/workspace/` are copied to container
- Common for package managers, shells, development tools

**Agents**: Standalone kits that define complete sandbox agents
- Require `kind: agent` in spec.yaml
- Must define credential sources for injected secrets
- Include service domains and authentication methods
- Examples: AMP coding agent, nanobot chat assistant

## Key Directories

- `spec/`: Kit artifact parsing and validation library
- `tck/`: Technology Compatibility Kit test framework
- `mixins/brew/`: Homebrew package manager mixin
- `mixins/code-server/`: Web VS Code with Claude Code extension mixin
- `amp/`, `nanobot/`, `nanoclaw/`, `openclaw/`, `pi/`, `trivy/`: Standalone agent kits
- `.github/`: CI workflows that validate kits on PR

## Best Practices

1. **Spec Validation**: Always run `sbx kit validate` before testing
2. **TCK Coverage**: Write comprehensive TCK tests that exercise all kit functionality
3. **Security**: Define minimal required network domains and credentials
4. **Reproducibility**: Pin kit versions using commit SHAs in production usage
5. **Documentation**: Keep kit README.md updated with usage examples
6. **Local Testing**: Validate kits run correctly in sandbox before submitting PR

This repository uses Docker Sandboxes' kit system to extend agent capabilities. Understanding the spec.yaml format and TCK requirements is essential for contributing effectively.