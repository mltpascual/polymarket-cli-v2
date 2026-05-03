# Polymarket CLI V2

Command-line client for Polymarket, patched for the CLOB V2 migration.

This fork keeps the original `polymarket` CLI interface while updating the CLOB client, order signing, collateral defaults, and approval checks for Polymarket's V2 exchange contracts.

> This is an independent fork, not an official Polymarket release. Use at your own risk and verify all transactions before signing.

## Features

- Browse Polymarket markets, events, tags, series, profiles, comments, sports metadata, and on-chain data.
- Query CLOB prices, spreads, midpoints, order books, markets, balances, orders, trades, rewards, and account status.
- Place, cancel, and manage CLOB orders through the V2 Rust SDK.
- Check pUSD and CTF token approvals against the V2 exchange contracts.
- Supports `table` and `json` output for both interactive and scripted usage.

## V2 Changes

This fork updates the official CLI for Polymarket CLOB V2:

- Replaces the legacy Rust SDK with `polymarket_client_sdk_v2`.
- Uses V2 order signing behavior from the V2 SDK.
- Uses pUSD as the default collateral token:
  - `0xC011a7E12a19f7B1f670d46F03B03f3342E82DFB`
- Checks approvals against the V2 contracts:
  - CTF Exchange V2: `0xE111180000d2663C0091e4f400237545B87B996B`
  - Neg Risk Exchange V2: `0xe2222d279d744050d28e00520010520000310F59`
  - Neg Risk Adapter: `0xd91E80cF2E7be2e162c6513ceD06f1dD0dA35296`
- Derives the configured trading wallet for `approve check`:
  - `proxy` checks the derived Polymarket proxy wallet
  - `gnosis-safe` checks the derived safe wallet
  - `eoa` checks the signer address directly
- Blocks `approve set` in `proxy` and `gnosis-safe` modes because direct EOA approval transactions cannot approve assets held by the derived wallet.

## Install

Build from source:

```bash
git clone <repo-url>
cd polymarket-cli-v2
cargo build --release
```

Install the binary somewhere on `PATH`:

```bash
install -m 0755 target/release/polymarket /usr/local/bin/polymarket
```

On Apple Silicon Homebrew installs, this path is commonly used instead:

```bash
install -m 0755 target/release/polymarket /opt/homebrew/bin/polymarket
```

Verify:

```bash
polymarket --version
```

Expected:

```text
polymarket 0.1.5-v2.1
```

## Configuration

The CLI resolves private keys in this order:

1. `--private-key`
2. `POLYMARKET_PRIVATE_KEY`
3. `~/.config/polymarket/config.json`

The default signature type is `proxy`.

```bash
polymarket wallet create
polymarket wallet import <private-key>
polymarket wallet show
```

Supported signature types:

- `proxy`
- `eoa`
- `gnosis-safe`

Override per command:

```bash
polymarket --signature-type eoa wallet show
```

## Safety

Do not commit or publish:

- `.env`
- private keys
- CLOB API credentials
- `~/.config/polymarket/config.json`
- build artifacts under `target/`

The `.gitignore` excludes common local files and Rust build output, but review `git status --short --ignored` before publishing a fork.

## Approvals

Check approvals:

```bash
polymarket approve check
```

In `proxy` and `gnosis-safe` modes, use the Polymarket web app to approve the derived wallet. This CLI intentionally blocks `approve set` in those modes to avoid sending approvals from the wrong address.

Direct EOA approvals are available only when explicitly requested:

```bash
polymarket --signature-type eoa approve set
```

Only use direct EOA approvals if you intend to trade directly from the EOA.

## Common Commands

Market browsing:

```bash
polymarket markets list --limit 10
polymarket markets search "bitcoin"
polymarket events list --limit 10
polymarket tags list
```

CLOB market data:

```bash
polymarket clob ok
polymarket clob book <TOKEN_ID>
polymarket clob price <TOKEN_ID> --side buy
polymarket clob midpoint <TOKEN_ID>
polymarket clob spread <TOKEN_ID>
```

Authenticated account reads:

```bash
polymarket clob orders
polymarket clob trades
polymarket clob balance --asset-type collateral
polymarket clob balance --asset-type conditional --token <TOKEN_ID>
polymarket clob account-status
```

Trading:

```bash
polymarket clob create-order --token <TOKEN_ID> --side buy --price 0.50 --size 10
polymarket clob market-order --token <TOKEN_ID> --side buy --amount 5
polymarket clob cancel <ORDER_ID>
polymarket clob cancel-all
```

On-chain data:

```bash
polymarket data positions 0xWALLET
polymarket data closed-positions 0xWALLET
polymarket data trades 0xWALLET
polymarket data leaderboard --period month --order-by pnl
```

JSON output:

```bash
polymarket -o json markets list --limit 5
polymarket -o json clob balance --asset-type collateral
```

## Development

```bash
cargo fmt --check
cargo check
cargo test
cargo build --release
```

Project layout:

```text
src/
  main.rs        CLI entry point
  auth.rs        Wallet resolution, RPC provider, CLOB authentication
  config.rs      Config file handling
  commands/      Command implementations
  output/        Table and JSON rendering
```

## Limitations

The following paths should be treated carefully and tested with small size first:

- live order placement
- live order cancellation
- direct EOA approval transactions
- CTF split, merge, and redeem transactions

## License

MIT
