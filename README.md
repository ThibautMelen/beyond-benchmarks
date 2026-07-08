# beyond-benchmarks

A leaderboard won't page you when a provider silently updates its weights.
This file will.

Three probes cut from production traffic. Four pinned models. Deterministic
oracles. A scoreboard on disk, diffed against the last run you trusted. Put it
on cron and silent regressions stop being silent.

Live-run at the [AI Tinkerers Paris](https://paris.aitinkerers.org/) closed-door
dinner [Beyond Benchmarks: Evaluating Models in Production](https://paris.aitinkerers.org/p/beyond-benchmarks-a-closed-door-dinner-on-evaluating-models-in-production)
on July 8th, 2026, sponsored by [OpenRouter](https://openrouter.ai). Written in
[Nika](https://nika.sh), a workflow language for AI where the file is audited
before it runs.

The grading path contains zero LLM calls. jq, not vibes.

```text
🦋 nika · beyond-benchmarks · 19 tasks
   permits ✓ declared boundary · default-deny

✔  llama_extract   infer · ollama/llama3.2:3b       3.0s ∥
✔  llama_triage    infer · ollama/llama3.2:3b       3.8s ∥
✔  llama_locale    infer · ollama/llama3.2:3b       4.5s ∥
✔  gemini_extract  infer · gemini/gemini-2.5-flash  1.4s ∥
✔  gemini_triage   infer · gemini/gemini-2.5-flash  2.0s ∥
✔  gemini_locale   infer · gemini/gemini-2.5-flash  1.5s ∥
✔  grok_extract    infer · xai/grok-3               2.8s · $0.001362 ∥
✔  grok_triage     infer · xai/grok-3               7.0s · $0.001215 ∥
✔  grok_locale     infer · xai/grok-3               2.5s · $0.001137 ∥
✔  gpt_extract     infer · openai/gpt-4o-mini       1.3s · $0.000034 ∥
✔  gpt_triage      infer · openai/gpt-4o-mini       1.2s · $0.000029 ∥
✔  gpt_locale      infer · openai/gpt-4o-mini       1.3s · $0.000024 ∥
✔  score           invoke · nika:jq                  0ms
✔  render_json     invoke · nika:jq                  0ms ∥
✔  write_json      invoke · nika:write               1ms ∥
✔  render_md       invoke · nika:jq                  0ms ∥
✔  write_md        invoke · nika:write               1ms ∥
✔  read_baseline   invoke · nika:read                0ms ∥
✔  drift           invoke · nika:jq                  0ms ∥
── 19/19 done · ≥ $0.003802 (6 unpriced) · elapsed 7.0s ────────
```

One verdict costs $0.004 and takes seven seconds.

## The dinner-night scoreboard

| model | score | failed probes |
|---|---|---|
| ollama/llama3.2:3b | 2/3 | incident_triage |
| gemini/gemini-2.5-flash | 3/3 | none |
| xai/grok-3 | 2/3 | incident_triage |
| openai/gpt-4o-mini | 3/3 | none |

Three conversations this table started:

**grok-3 ate 97% of the metered cost and lost to a model 40x cheaper.**
It read a full EU payments outage and answered `"severity": "P2"`. Defensible.
Wrong for us. The oracle encodes your severity bar, not the model's opinion.

**llama3.2:3b parsed the French comma-decimal, then invented a date.**
`1 834,50 €` came back as `1834.5`, correct. The payment deadline came back as
`"2024-04-01"`, a date that appears nowhere in the email. Small models fail per
capability, not per size.

**The drift sentinel fired 46 seconds after it was armed.** Before temperature
was pinned to 0, two consecutive runs on the same machine flipped the locale
probe. Pure sampling variance. Picture a silent weight update on a provider
carrying 30% of your traffic:

```json
{"status": "DRIFT DETECTED",
 "changes": [{"model": "ollama/llama3.2:3b", "was": "1/3", "now": "2/3",
              "probes_now_failing": ["locale_money"]}]}
```

## The audit, before a single token

`nika check` prices every task, verifies the permit boundary, and traces
information flow. Statically. Refusal is cheaper than regret.

```text
 ✔ PLAN     4 wave(s) · 19 task(s) · max parallelism 13
   grok_extract  xai/grok-3  ≤400 tk  $0.0060
   gpt_extract  openai/gpt-4o-mini  ≤400 tk  $0.0002
 ✔ SECRETS  no information-flow escapes
 ✔ PERMITS  body fits the declared boundary
 ✔ audited · 19 task(s) · 4 wave(s) · permits declared · est ≥$0.0187 · 0 hints
```

The permits are default-deny: this file may read `baseline.json`, write the two
scoreboard files, and call four named builtins. Nothing else. `exec` is off.
Local models are marked unpriced with a `≥` floor instead of a fake `$0.00`.

## Run it

```bash
brew install supernovae-st/homebrew-nika/nika
ollama pull llama3.2:3b
export GEMINI_API_KEY=... XAI_API_KEY=... OPENAI_API_KEY=...

nika check beyond-benchmarks.nika.yaml
nika run beyond-benchmarks.nika.yaml --max-cost-usd 0.25
cat scoreboard.md
```

`--max-cost-usd` blocks before the call that would cross the budget, not after.

## Arm the sentinel

```bash
cp scoreboard.json baseline.json
nika run beyond-benchmarks.nika.yaml --max-cost-usd 0.25
```

Every run now ends with a verdict: scores match the baseline, or a diff naming
the model, the probe, and both scores. Cron the run. That verdict is your pager.

## No keys, no network

```bash
nika test beyond-benchmarks.nika.yaml
```

Runs the whole DAG under the mock provider against a committed golden file.
Deterministic, offline, $0.00.

## Make it yours

- **Probes.** Paste your own traffic snippet into `vars:`, give the task a
  schema and a one-line jq oracle. Probes are cheap rows; the discipline is
  one deterministic oracle per probe.
- **Roster.** Each model is one pinned task block. Candidates parked in the
  file: `mistral/mistral-small-latest`, `openrouter/auto`, anthropic. Add a
  key, copy a block. The catalog has 38 providers.
- **Cadence.** The run is idempotent and every run leaves a tamper-evident
  trace (`nika trace verify`). Your eval dataset accumulates for free.

## Links

- Event · [Beyond Benchmarks, AI Tinkerers Paris](https://paris.aitinkerers.org/p/beyond-benchmarks-a-closed-door-dinner-on-evaluating-models-in-production)
- Community · [paris.aitinkerers.org](https://paris.aitinkerers.org/)
- Sponsor · [OpenRouter](https://openrouter.ai)
- Nika · [nika.sh](https://nika.sh) · [docs](https://docs.nika.sh) · [engine](https://github.com/supernovae-st/nika) · [spec](https://github.com/supernovae-st/nika-spec) · [brew tap](https://github.com/supernovae-st/homebrew-nika)

## License

Apache-2.0, same as the Nika workflow spec. The scoreboards committed here are
the real dinner-night outputs, kept as evidence.
