# beyond-benchmarks

**The benchmark that rides production.** One auditable workflow file: probes cut
from real traffic, graded by deterministic oracles, cost-capped *before* a single
token is spent, and diffed against the last known-good run.

Built for — and live-run at — the [AI Tinkerers Paris](https://paris.aitinkerers.org/)
closed-door dinner [**Beyond Benchmarks: Evaluating Models in Production**](https://paris.aitinkerers.org/p/beyond-benchmarks-a-closed-door-dinner-on-evaluating-models-in-production)
(Paris, July 8th 2026 · sponsored by [OpenRouter](https://openrouter.ai)).

Written in [Nika](https://nika.sh) — a workflow language for AI where the file is
audited before it runs (`nika: v1` · [spec](https://github.com/supernovae-st/nika-spec) Apache-2.0 ·
[engine](https://github.com/supernovae-st/nika) AGPL-3.0 · [docs](https://docs.nika.sh)).

## Why

Leaderboard performance is increasingly disconnected from real-world reliability
— the dinner's exact premise. This file answers its four table themes with
running code instead of slides:

| Dinner theme | What the file does |
|---|---|
| **Live evaluation systems** | the probes ARE production traffic snippets; every run leaves a tamper-evident trace (`nika trace verify`) — your eval dataset accumulates for free |
| **Routing as a primitive** | the model roster is explicit, pinned, reviewable — 4 providers in one file; adding `openrouter/auto` or any of 38 catalog providers is one block |
| **Model drift in practice** | `baseline.json` + a diff task = a drift sentinel; cron the run and silent regressions page you |
| **Cost vs. reliability** | `nika check` prints a per-task price ladder before any spend; `--max-cost-usd` blocks *before* the call that would cross it; `≥` markers refuse the local-is-free lie |

## The scoreboard it produced on dinner night

Four models × three probes (FR-locale invoice extraction · incident triage ·
comma-decimal money parsing), graded by jq oracles — **zero LLM in the grading
path**. Run on 2026-07-08, temperature pinned to 0:

| model | score | failed probes |
|---|---|---|
| ollama/llama3.2:3b | 2/3 | incident_triage |
| gemini/gemini-2.5-flash | 3/3 | — |
| xai/grok-3 | 2/3 | incident_triage |
| openai/gpt-4o-mini | 3/3 | — |

Three real exhibits from that run:

- **grok-3** — priced ~40× gpt-4o-mini and ~95% of the run's metered cost —
  called a full EU payments outage `"severity": "P2"`. Defensible, but the
  oracle encodes *your* severity bar, not the model's opinion.
- **llama3.2:3b** nailed the French comma-decimal total (`1 834,50 €` → `1834.5`)
  but hallucinated `due_date: "2024-04-01"` — a date that appears nowhere in the
  email. Failure modes are per-capability, not per-model-size.
- Before temperature was pinned, the drift sentinel caught **llama3.2:3b flipping
  a probe between two runs 46 seconds apart** — same file, same machine, pure
  sampling variance. If two consecutive runs do that, imagine a silent
  provider-side weight update.

## Run it

```bash
brew install supernovae-st/homebrew-nika/nika   # or see https://docs.nika.sh
ollama pull llama3.2:3b                          # the local leg (zero key)
export GEMINI_API_KEY=... XAI_API_KEY=... OPENAI_API_KEY=...

nika check beyond-benchmarks.nika.yaml           # audit: waves · permits · cost ladder
nika run beyond-benchmarks.nika.yaml --max-cost-usd 0.25
cat scoreboard.md
```

The whole run costs well under a cent metered (~$0.004 on dinner night, grok-3
being most of it) and takes ~5 seconds.

Arm the drift sentinel, then re-run any time (cron it — that's the point):

```bash
cp scoreboard.json baseline.json
nika run beyond-benchmarks.nika.yaml --max-cost-usd 0.25
# → drift: "no drift — every model scores exactly as baselined"
#   ...until a provider silently changes something under you.
```

### No keys / no network

```bash
nika test beyond-benchmarks.nika.yaml   # mock provider · golden-pinned · $0.00
```

### What the audit shows before a token is spent

```text
 ✔ PLAN     4 wave(s) · 19 task(s) · max parallelism 13
 ⚠ COST     $0.0187 FLOOR — per-task ladder · unpriced local marked, never $0.00-faked
 ✔ SECRETS  no information-flow escapes
 ✔ PERMITS  body fits the declared boundary   (fs: 1 read · 2 writes · exec: false)
```

## Make it yours

- **Swap the probes** — paste your own traffic snippet into `vars:`, give the
  task a schema and a one-line jq oracle. The discipline is one deterministic
  oracle per probe; probes are cheap rows.
- **Swap the roster** — each model is one pinned task block. Parked candidates
  are noted in the file (mistral, `openrouter/auto`, anthropic — add a key, copy
  a block).
- **Cron it** — the run is idempotent, the trace is append-only, the drift task
  is your pager.

## Links

- Event · [Beyond Benchmarks — AI Tinkerers Paris](https://paris.aitinkerers.org/p/beyond-benchmarks-a-closed-door-dinner-on-evaluating-models-in-production)
- Community · [AI Tinkerers Paris](https://paris.aitinkerers.org/)
- Sponsor · [OpenRouter](https://openrouter.ai)
- Nika · [nika.sh](https://nika.sh) · [docs.nika.sh](https://docs.nika.sh) · [engine](https://github.com/supernovae-st/nika) · [spec](https://github.com/supernovae-st/nika-spec) · [install](https://github.com/supernovae-st/homebrew-nika)

## License

Apache-2.0 — same as the Nika workflow spec. The scoreboards committed here are
the real outputs from dinner night, kept as evidence.
