# ldk-server-agent-wallet

Reusable isolated wallet bundle for running `ldk-server` behind `ldk-server-mcp`.

This repository is meant to be copied or cloned into a separate directory, populated
with locally built binaries, configured for a specific node, and launched via
`./run-ldk-server-mcp`.

## What this repo contains

- `config.example.toml` - template `ldk-server` config for a standalone wallet
- `run-ldk-server-mcp` - wrapper that starts `ldk-server`, waits for TLS/API key
  generation, exports MCP environment variables, then starts `ldk-server-mcp`
- `SKILL.md` - instructions another agent can follow to reproduce the setup
- `CLAUDE.md` - repository-specific maintenance notes

## Upstream repositories

- `ldk-server`: `https://github.com/lightningdevkit/ldk-server`
- `ldk-server-mcp`: `https://github.com/tnull/ldk-server-mcp`

## Directory layout

The runtime bundle expects these local paths:

```text
.
├── bin/
│   ├── ldk-server
│   ├── ldk-server-cli
│   └── ldk-server-mcp
├── data/
├── run/
├── config.example.toml
└── run-ldk-server-mcp
```

`bin/`, `data/`, and `run/` are kept empty in git except for `.gitkeep` placeholders.

## Setup

1. Build the upstream binaries.
2. Copy the resulting binaries into `./bin/`:
   - `ldk-server`
   - `ldk-server-cli`
   - `ldk-server-mcp`
3. Copy `config.example.toml` to `config.toml`.
4. Edit `config.toml`:
   - set an Esplora server, or keep the default `https://mempool.bitcoin.ninja/api`
   - if LSPS2 client details are available, uncomment and fill in the
     `[liquidity.lsps2_client]` section
   - if LSPS2 client details are not available, leave that section commented out
5. Start the MCP entrypoint:

```bash
./run-ldk-server-mcp
```

## Building upstream binaries

### `ldk-server`

```bash
git clone https://github.com/lightningdevkit/ldk-server
cd ldk-server
cargo build --release -p ldk-server -p ldk-server-cli
```

### `ldk-server-mcp`

```bash
git clone https://github.com/tnull/ldk-server-mcp
cd ldk-server-mcp
cargo build --release
```

Then install the binaries into this repo:

```bash
install -m 755 /path/to/ldk-server/target/release/ldk-server ./bin/ldk-server
install -m 755 /path/to/ldk-server/target/release/ldk-server-cli ./bin/ldk-server-cli
install -m 755 /path/to/ldk-server-mcp/target/release/ldk-server-mcp ./bin/ldk-server-mcp
```

## Validation

After starting `./run-ldk-server-mcp`, check that it responds over stdio:

```bash
printf '%s\n' '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | ./run-ldk-server-mcp
```

You should get a JSON-RPC response listing the exposed MCP tools.

## Configuration defaults

- Default Esplora URL: `https://mempool.bitcoin.ninja/api`
- LSPS2 client configuration: optional; keep it commented out unless the user provides:
  - `node_pubkey`
  - `address`
  - optional `token`
