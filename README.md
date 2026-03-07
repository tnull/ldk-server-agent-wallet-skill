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

- `ldk-server`: `https://github.com/tnull/ldk-server/tree/2026-03-lsps2-client-support`
- `ldk-server-mcp`: `https://github.com/tnull/ldk-server-mcp/tree/2026-03-lsps2-client-support`

These branch references are temporary and should be used while the LSPS2 client-support
work has not yet landed on the default branches.

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
    - confirm the user really wants to run on mainnet before keeping `network = "bitcoin"`
    - if the user wants `signet` or `mutinynet`, set `network = "signet"`
    - set `storage.disk.dir_path` to the data directory the user wants to use
    - if no data directory is provided, default to `~/.ldk-server-agent-wallet/data`
    - set an Esplora server, or keep the default for the selected network:
      - mainnet: `https://mempool.bitcoin.ninja/api`
      - signet / mutinynet: `https://mutinynet.com/api`
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
git clone --branch 2026-03-lsps2-client-support https://github.com/tnull/ldk-server
cd ldk-server
cargo build --release -p ldk-server -p ldk-server-cli
```

### `ldk-server-mcp`

```bash
git clone --branch 2026-03-lsps2-client-support https://github.com/tnull/ldk-server-mcp
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

- Mainnet safety check: agents should explicitly confirm that the user wants `network = "bitcoin"`
- Default data directory: `~/.ldk-server-agent-wallet/data`
- Default Esplora URL on mainnet: `https://mempool.bitcoin.ninja/api`
- Default Esplora URL on signet / mutinynet: `https://mutinynet.com/api`
- LSPS2 client configuration: optional; keep it commented out unless the user provides:
  - `node_pubkey`
  - `address`
  - optional `token`
