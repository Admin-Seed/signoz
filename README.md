# signoz — deployment configuration for seed-vm-services-eastus

**This is not a fork.** It contains no SigNoz source code and has no upstream
remote. Nothing here ever needs to be synced or merged.

It previously held a full fork of the SigNoz monorepo (202 MB) purely to carry
one extra container and ~700 bytes of Azure Monitor config. SigNoz removed
`deploy/docker/` from its repository at **v0.130.0** — the bundled Compose files
are deprecated and no longer distributed — and replaced them with
[Foundry](https://github.com/SigNoz/foundry), which generates them from a short
declarative file. That removed the last reason to carry a fork.

The pre-migration fork is preserved at tag `archive/fork-2026-09-11`.

## Layout

| Path | What it is |
| --- | --- |
| `signoz/casting.yaml` | **The only file you edit.** Foundry input. |
| `signoz/casting.yaml.lock` | Checksums of the generated output. Committed. |
| `signoz/pours/deployment/` | `foundryctl forge` output. Generated — do not hand-edit. |
| `azure-collector/` | Standalone Azure Monitor collector, deployed as its own stack. |

## Upgrading SigNoz

```sh
# 1. bump the image tags in signoz/casting.yaml
# 2. regenerate
cd signoz && foundryctl forge -f casting.yaml
# 3. commit and push; the Komodo webhook redeploys
```

Check the [upgrade guides](https://signoz.io/docs/operate/upgrade/) first — some
releases carry one-way migrations (v0.131 ClickHouse 25.12.5, v0.135
Dashboards V2, v0.137 saved views).

## Secrets

None are stored here. Komodo holds them and writes a `.env` beside the Compose
file at deploy time:

| Stack | Variables |
| --- | --- |
| `signoz` | `SIGNOZ_TOKENIZER_JWT_SECRET`, `COMPOSE_PROJECT_NAME` |
| `signoz-azure-collector` | `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_SUBSCRIPTION_ID` |

## State

Two Docker volumes carry everything that matters and are re-attached by name via
patches in `casting.yaml`:

- **`signoz-sqlite`** — dashboards, users, TTL settings. Small and irreplaceable.
- **`signoz-clickhouse`** — telemetry. 10-day TTL, so effectively disposable.

Changing those patched names starts SigNoz with an empty database **and reports
no error**. Do not touch them.
