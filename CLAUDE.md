# CLAUDE.md

This repository is a reusable runtime bundle for `ldk-server` plus `ldk-server-mcp`.

## Purpose

- Provide a clean, isolated folder layout for an agent-controlled Lightning wallet.
- Document how to build and install the required upstream binaries.
- Keep a reusable launcher script checked in so future agents can bootstrap the same setup.

## Repository layout

- `config.example.toml` - template `ldk-server` configuration
- `run-ldk-server-mcp` - launcher that starts `ldk-server` if needed, exports MCP env vars, and execs `ldk-server-mcp`
- `SKILL.md` - step-by-step agent workflow for reproducing the setup
- `bin/` - local binary install target, ignored in git except for `.gitkeep`
- `data/` - runtime state directory, ignored in git except for `.gitkeep`
- `run/` - pid and lock files, ignored in git except for `.gitkeep`

## Important conventions

- Do not commit real `config.toml`, TLS material, API keys, or database files.
- Always explicitly confirm that the user really wants to run on mainnet before leaving `network = "bitcoin"` in the config.
- If the user wants signet or mutinynet, set `network = "signet"` and default Esplora to `https://mutinynet.com/api` unless they provide a different server.
- Ask the user for the runtime data directory. If they do not provide one, default to `~/.ldk-server-agent-wallet/data`.
- Ask the user for an Esplora server. If they do not provide one, default to `https://mempool.bitcoin.ninja/api` on mainnet or `https://mutinynet.com/api` on signet / mutinynet.
- Ask the user for LSPS2 client details. If they do not provide them, leave the `[liquidity.lsps2_client]` section commented out.
- Upstream repositories to use:
  - `https://github.com/tnull/ldk-server/tree/2026-03-lsps2-client-support`
  - `https://github.com/tnull/ldk-server-mcp/tree/2026-03-lsps2-client-support`
- These branch references are the current LSPS2-capable sources and should be preferred until the work lands on default branches.

## Common commands

```bash
cp config.example.toml config.toml
./run-ldk-server-mcp
```
