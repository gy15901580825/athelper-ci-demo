# ATHelper CI Demo

> A live, public demo of how ATHelper red-team probes get wired into a customer's CI/CD pipeline.

This is **not** a real customer's agent — it's a synthetic project that points ATHelper at a deliberately-vulnerable demo chatbot we host at `demo-real.at-helper.com` (Llama 3.3 70B on Azure Foundry, deliberately weak system prompt). The CI workflow runs the full ATHelper probe library against it and uploads the SARIF result to GitHub's Security tab.

The point: prospects see what their own repo would look like with ATHelper integrated, without exposing their production agent.

---

## What you can see in this repo

1. **Workflow file** — [`.github/workflows/athelper-probe.yml`](.github/workflows/athelper-probe.yml). 30 lines. Does install → run → upload SARIF.

2. **Target spec** — [`demo-real-target.json`](demo-real-target.json). Tells ATHelper how to reach the agent (URL, request shape, auth headers, JSONPath for response).

3. **Live runs** — [Actions tab](../../actions). Manually trigger or wait for the Monday cron.

4. **Findings** — [Security → Code scanning](../../security/code-scanning). Each finding shows up as a row with severity, attack prompt excerpt, and target response, mapped to OWASP LLM Top 10 + MITRE ATLAS.

---

## Run the demo

### Option A: Manual trigger (most demo-friendly)

1. Go to [Actions → athelper-probe → Run workflow](../../actions/workflows/athelper-probe.yml).
2. Pick `probes: all` (or a subset like `owasp_07_system_prompt_leakage`).
3. Cost cap defaults to `$5.00` (full sweep ≈ $2.20).
4. Watch the run; ~5–15 minutes for a full sweep.
5. Open [Security → Code scanning](../../security/code-scanning) — findings appear as alerts.

### Option B: Local repro

```bash
# 1. Install CLI
curl -sSL https://github.com/gy15901580825/at_helper_cli/releases/download/v0.1.1/athelper-probe-linux-x86_64 \
  -o /usr/local/bin/athelper-probe
chmod +x /usr/local/bin/athelper-probe

# 2. Set token
export ATHELPER_API_TOKEN=...   # email contact@veyon.solutions for one

# 3. Run
athelper-probe run \
  --target demo-real-target.json \
  --probes all \
  --per-run-cap 5.00 \
  --format sarif --out report.sarif
```

---

## Wiring this into your own repo

Three edits:

1. Replace `demo-real-target.json` with your own `target.json` pointing at your agent
2. Add a GitHub repository secret: `ATHELPER_API_TOKEN`
3. Copy `.github/workflows/athelper-probe.yml`, optionally adding a `pull_request:` trigger to scan every PR

That's it. SARIF uploads to your Security tab automatically alongside CodeQL / Snyk / Dependabot.

---

## What you should expect to see

A typical run against the demo-real Llama target produces ~115 findings (≈ 35 probes are auto-skipped because they're browser-only and this target is HTTP):

| Verdict | Count | Meaning |
|---|---|---|
| ✅ pass | ~80 | Llama refused the attack |
| ⏭ skipped | ~30 | Probe target_class incompatible with HTTP target |
| 🟡 warn | 1–3 | Llama partially complied (judge confidence < 1.0) |
| 🛡 blocked_by_target | 0–5 | Foundry's Prompt Shield caught the prompt before it reached Llama |
| 🔴 fail | 0–2 | Llama leaked something the rubric considers a failure |

Run-to-run variance is normal — Llama is non-deterministic and so are the iterative attacks (TAP/PAIR/GCG).

---

## Notes for sales conversations

- **Why a hosted demo target instead of testing prospect's agent live**: prospects don't want to hand over an agent endpoint mid-call. This lets them see the report shape against a comparable surface (HTTP chat agent) without that friction.
- **Why Llama 3.3 70B specifically**: Meta's safety training is well-respected; if ATHelper finds anything against this baseline, it's signal. Azure Foundry's Prompt Shield is a real production-grade defense; the demo shows what defense-in-depth looks like in findings.
- **Two demo targets exist**:
  - `demo-real.at-helper.com` (this repo's target) — real Llama, honest results
  - `demo.at-helper.com` — deterministic-leak version, always shows critical findings; useful for the 30-second elevator pitch

---

## Related

- ATHelper docs: [at-helper.com/docs](https://www.at-helper.com/docs)
- Become a design partner: [at-helper.com/design-partners](https://www.at-helper.com/design-partners)
- Pricing: [at-helper.com/pricing](https://www.at-helper.com/pricing)
- ATHelper CLI source: [gy15901580825/at_helper_cli](https://github.com/gy15901580825/at_helper_cli)
