# Environment Setup

Everything you need installed to use Scott's tech stack. Run through this on any new machine.

## Required Tools

| Tool | Version | What It Does |
|------|---------|-------------|
| Node.js | v22 LTS | JavaScript runtime (needed for Nuxt, npm) |
| npm | v10+ | Package manager (comes with Node) |
| Rust | latest stable | Backend language (needed for Tauri) |
| Cargo | latest stable | Rust package manager (comes with Rust) |
| SurrealDB | v3.0.0 | Multi-model database |
| Tauri CLI | v2.x | Desktop app framework CLI |
| Git | latest | Version control |
| Claude Code | latest | AI development tool |

## Installation Commands (macOS)

### Node.js and npm
```bash
# Install via Homebrew
brew install node

# Verify
node --version   # Should show v22.x
npm --version    # Should show v10.x
```

### Rust and Cargo
```bash
# Install via rustup (official installer)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Restart terminal, then verify
rustc --version  # Should show latest stable
cargo --version
```

### SurrealDB
```bash
# Install via Homebrew
brew install surrealdb/tap/surreal

# Verify
surreal version  # Should show 3.0.0+

# Start a local server (for web app development)
surreal start --user root --pass root --bind 0.0.0.0:8000 file:./mydata.db
```

### Tauri CLI
```bash
# Install the Tauri CLI globally
cargo install tauri-cli

# Verify
cargo tauri --version  # Should show 2.x
```

### Additional macOS Dependencies (for Tauri)
```bash
# Xcode command line tools (if not already installed)
xcode-select --install
```

## Verification Checklist

Run these commands to confirm everything is working:

```bash
echo "=== Node ===" && node --version
echo "=== npm ===" && npm --version
echo "=== Rust ===" && rustc --version
echo "=== Cargo ===" && cargo --version
echo "=== SurrealDB ===" && surreal version
echo "=== Tauri CLI ===" && cargo tauri --version
echo "=== Git ===" && git --version
```

All commands should return version numbers without errors.

## VS Code Extensions (Recommended)

| Extension | Why |
|-----------|-----|
| Vue - Official | Vue/Nuxt syntax highlighting and IntelliSense |
| Tailwind CSS IntelliSense | Tailwind class autocomplete |
| rust-analyzer | Rust language support |
| Tauri | Tauri-specific tooling |
| SurrealQL | SurrealDB query syntax highlighting |
| Error Lens | Inline error display |

## Claude Code Setup Notes

Claude Code should already be installed. To configure it for this toolkit:

1. The global `~/.claude/CLAUDE.md` should reference the toolkit path
2. Each project's `CLAUDE.md` should list which specific skill files to read
3. MCP servers are configured per-machine (see `mcp-config/mcp-setup-guide.md`)

## Scott's Custom Scripts

| Script | Location | What It Does |
|--------|----------|-------------|
| Start SurrealDB | `~/Sites/start-surreal.sh` | Starts local SurrealDB server with standard config |

## Machine Sync

Both machines (Mac Studio + MacBook Air) sync via GitHub:
```bash
# Pull latest toolkit changes
cd ~/scott-toolkit && git pull

# After making changes, push
cd ~/scott-toolkit && git add -A && git commit -m "description" && git push
```
