## Wallet Tracker — real-time Solana wallet analyzer

No fluff. Streams from Yellowstone gRPC. On-tip metrics. One binary. Prints a clean table, or NDJSON for bots. Optional Prometheus at /metrics.

### Why it wins
- **On-tip**: Streams transactions via Yellowstone (no polling)
- **Fast**: Zero allocations hot path, compact state, smart pruning
- **Signal-first**: Fee-only and micro-delta filters keep noise out
- **Wallet IQ**: PnL, win/loss, winrate, volume, fees, tx_60s, counterparties, SPL positions
- **Program aware**: System / Token / Other counts per wallet
- **Hydrates itself**: `from_slot` backfills then stays live; auto-deepen if needed
- **Bot-ready**: NDJSON (compact/full), selectable fields
- **Prometheus**: Export gauges for dashboards
- **Resilient**: Multi-endpoint with health scoring + rotation, non-blocking webhooks

### Configure
Create `config.yaml` (all options are optional; CLI/env can override):

```yaml
ep endpoint: https://your.yellowstone.endpoint:443
x_token: ${YELLOWSTONE_XTOKEN}
wallets: [ "Fh2…abcd", "7Gv…wxyz" ]
window_secs: 604800
commitment: confirmed
max_msg_mb: 64
allow_failed: false
summary_secs: 5
output_ndjson: false
ndjson_profile: full        # compact|full
ndjson_fields: [wallet, pnl, trades, tx60]  # only used when profile=compact
webhook_url: null
webhook_buffer: 1000
from_slot: null
min_delta_lamports: 50000
exclude_fee_only: true
enable_gzip: false
enable_zstd: false
alert_pnl_up_sol: null
alert_pnl_down_sol: null
auto_replay_deepen: true
replay_deepen_slots: 2000000
deepen_ticks: 10
endpoints: []                # optional alternates for failover
metrics_bind: "0.0.0.0:9464" # serve /metrics (Prometheus)
no_color: false
```

### Run
```bash
cargo build --release
./target/release/wallet-tracker --config config.yaml
```

Direct flags (override config):
- `--endpoint`, `--x-token`, `--wallets a,b,c`, `--window-secs`, `--summary-secs`
- `--output-ndjson`, `--from-slot`, `--metrics-bind 0.0.0.0:9464`, `--no-color`
- `--ndjson-fields wallet,pnl,tx60` (when `ndjson_profile=compact`)

### Output
- Terminal: one-line header + dense table per wallet, with colored PnL, ΔPnL, winrate, fees, tx_60s, counterparties, top SPL, and a live trend sparkline.
- NDJSON: compact or full objects every tick (for bots/webhooks).
- Prometheus: `/metrics` exposes per-wallet gauges and endpoint health counters.

### Tuning
- Hydration: set `from_slot` near your lookback; auto-deepen guards empty token books
- Noise: raise `min_delta_lamports`, keep `exclude_fee_only: true`
- Throughput: tune `max_msg_mb`; add `endpoints` for resilience
- Alerts: set `alert_pnl_up_sol` / `alert_pnl_down_sol` with `webhook_url` 