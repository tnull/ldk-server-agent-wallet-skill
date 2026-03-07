# SKILL: Set up an isolated `ldk-server` wallet bundle

Use this workflow when a user wants a standalone folder that runs `ldk-server` via
`ldk-server-mcp`, with local binaries and runtime state isolated from other wallets.

## Goal

Create a self-contained bundle with:

- `ldk-server`
- `ldk-server-cli`
- `ldk-server-mcp`
- a local `config.toml`
- a reusable `run-ldk-server-mcp` launcher

## Upstream repositories

- `ldk-server`: `https://github.com/lightningdevkit/ldk-server`
- `ldk-server-mcp`: `https://github.com/tnull/ldk-server-mcp`

## Questions to ask the user

Ask for:

1. Whether they really want to run on mainnet.
2. The target directory for the wallet bundle.
3. The data directory the wallet should use.
4. The Esplora server URL.
5. Optional LSPS2 client details:
   - `node_pubkey`
   - `address`
   - optional `token`

## Defaults

- Do not assume mainnet silently. Require explicit confirmation before leaving:

```toml
[node]
network = "bitcoin"
```

- If the user does not provide a data directory, set:

```toml
[storage.disk]
dir_path = "~/.ldk-server-agent-wallet/data"
```

- If the user does not provide an Esplora server, set:

```toml
[esplora]
server_url = "https://mempool.bitcoin.ninja/api"
```

- If the user does not provide LSPS2 client details, leave the LSPS2 section disabled:

```toml
#[liquidity.lsps2_client]
#node_pubkey = "<lsp node pubkey>"
#address = "127.0.0.1:9735"
#token = ""
```

## Expected repository layout

```text
wallet-dir/
├── bin/
│   ├── ldk-server
│   ├── ldk-server-cli
│   └── ldk-server-mcp
├── data/
├── run/
├── config.toml
└── run-ldk-server-mcp
```

## Procedure

1. Clone or use local checkouts of the two upstream repositories.
2. Build `ldk-server` and `ldk-server-cli` from `ldk-server`.
3. Build `ldk-server-mcp` from `ldk-server-mcp`.
4. Install the three binaries into `wallet-dir/bin/`.
5. Copy `config.example.toml` to `config.toml`.
6. Confirm whether mainnet is really intended before keeping `network = "bitcoin"`.
7. Fill in the chosen data directory, or default to `~/.ldk-server-agent-wallet/data`.
8. Fill in the Esplora URL.
9. If the user supplied LSPS2 settings, uncomment and fill in `[liquidity.lsps2_client]`.
10. If not, leave `[liquidity.lsps2_client]` commented out.
11. Start the bundle through `./run-ldk-server-mcp`.
12. Validate with a `tools/list` JSON-RPC request.

## Validation command

```bash
printf '%s\n' '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | ./run-ldk-server-mcp
```

## Notes

- `run-ldk-server-mcp` automatically starts `ldk-server` if it is not already running.
- The wrapper waits for the generated TLS cert and API key, then exports:
  - `LDK_BASE_URL`
  - `LDK_API_KEY`
  - `LDK_TLS_CERT_PATH`
- The script derives its root directory from its own location, so it can be reused in any folder.
