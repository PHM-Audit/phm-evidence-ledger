# PHM Evidence Ledger — 5,000,086,001 (≈5B) measurement units, open artifacts

The aggregate record of the PHM deterministic-verification test campaigns,
closing state 2026-10-09, packaged 2026-10-11. Every number below is backed
by an on-repo artifact file; scripts are closed-source core assets (per
release discipline), metrics and audit logs are open.

> Total: **5,000,086,001 units** (precise). Caliber note: three layers count
> program/scene executions ("次"), the robotics layer counts data frames
> ("帧") — units are mixed by design and stated per line below.

## Four layers

| # | Layer | Count | Unit | Result fields (verbatim from artifact) | Artifact |
|---|---|---|---|---|---|
| 1 | Master 2B — physics-engine gate | 1,900,000,000 programs (+100,000,000 baseline) | program executions | TP=1,357,143,400; **FP=0; FN=0** (no false positives / false negatives observed in this test set and caliber); 10 classes × 135,714,340; layers L1=995,239,000 / L2=588,095,220 | `metrics/master_2b.json` + `logs/grind2b.log` (contains `REACHED_2B` closing mark) + `marks/master_2b_done` |
| 2 | Master Verifier — validation scenes | 1,000,000,000 | scene executions | per-class n/ok recorded verbatim in artifact: grounded 166,667,000/166,667,000; derived 166,667,000/166,667,000; structural 166,667,000/166,667,000; ungrounded 166,667,000/165,982,892; mismatch 166,666,000/166,666,000; phase_flip 166,666,000/166,666,000 (class semantics defined by the closed-source verifier) | `metrics/master_verifier_1b.json` + `logs/gv1b.log` |
| 3 | Robotics stream gate | 1,000,086,001 frames (260,892,000 sequences) | data frames | benign 43,482,000/43,482,000 PASS; **collision: 43,440,642/43,482,000 BLOCKed**; joint-limit and other adversarial classes per artifact (the gate actively blocks, not just scores) | `metrics/master_robotics.json` + `logs/grind_robotics.log` |
| 4 | SCL fuzzing | 1,000,000,000 programs | program executions | **crash=0** (no crash observed in this round); hits=3,226,364,783 (rule-hit breakdown in artifact: R5=1,340,454,673 / R1b=1,130,118,900 / R6=613,284,859 / …); avg 2.41 ms/program (full-recompute estimate ≈3,347 CPU-hours per README) | `metrics/fuzz_metrics.json` + `logs/` + `marks/fuzz_done` (OK) |

## How to verify (without trusting this README)

1. Read each `metrics/*.json` — the headline fields (`programs`, `n`, `tp/fp/fn`, `crash`, `n_frame`) are the source of truth; this README only restates them.
2. `logs/grind2b.log` contains the `REACHED_2B` closing mark; `logs/daemon_guard.log` (82,844 bytes) holds the guard-cycle audit trail; `marks/` holds completion flags.
3. Integrity: `SHA256SUMS.txt` covers every artifact in this repo.
4. Full-recompute cost estimate: SCL layer alone ≈3,347 CPU-hours (2.41 ms × 1e9); the other layers are in the same order of magnitude.

## Honest scope

- Raw per-execution records were **not** retained (≈50B executions); the
  on-disk assets are metrics JSON + audit logs + closing marks (~60 MB total).
  Re-execution from the closed scripts, or reading the JSON fields against
  the log marks, are the two verification paths.
- Zero-valued results (FP=0, FN=0, crash=0) mean "not observed in this test
  set and caliber" — they are not absolute guarantees.
- The three execution-count layers and the frame-count layer use different
  units; the 5,000,086,001 total mixes them deliberately and transparently.
- Scripts are closed-source core assets; only metrics, logs, and marks are
  published here.

## Related

- Method lineage: [agent-action-verifier](https://github.com/PHM-Audit/agent-action-verifier) (AgentDojo-validated deterministic verifier)
- Harness-native plugin: [dsh-action-guard](https://github.com/PHM-Audit/dsh-action-guard)
- Domain lines: [phm-unitree-guardrail](https://github.com/PHM-Audit/phm-unitree-guardrail) · [phm-drone-audit-cloud](https://github.com/PHM-Audit/phm-drone-audit-cloud) · [phm-deepseek-guardrail](https://github.com/PHM-Audit/phm-deepseek-guardrail)

## License

MIT for the ledger content; underlying scripts remain closed.

---

**PHM evidence chain** — aggregate ledger: [phm-evidence-ledger](https://github.com/PHM-Audit/phm-evidence-ledger) (5,000,086,001 units, four layers, open metrics + SHA-256 manifest) · method parent: [agent-action-verifier](https://github.com/PHM-Audit/agent-action-verifier) (AgentDojo-validated) · [PHM-Audit profile](https://github.com/PHM-Audit)


## Rerun 2026-10-16 — hallucination suppression & agent-interception (fresh, local-deterministic)

Three-line rerun executed 2026-10-16 (all local-deterministic, no LLM API in the loop; core engine
closed-source per release discipline — only verdicts, counters and an open driver script are published):

| Line | Scale | Result | Artifact |
|---|---|---|---|
| Agent action interception (7 rule families, synthetic adversarial flows) | 20,000 scenarios (14,000 attack + 6,000 legitimate) | intercept_rate = 1.0 (0 miss), false_intercept_rate = 0.0, worst 304 ms | `rerun_2026-10-16/action_audit_metrics.json` (audit_sha256 861c2c74c2d89741) |
| Concurrent tool-call injection stress (openai-agents SDK 0.23.1 internal executor path) | serial + concurrent + high-risk group, n=30 each | veto_rate = 1.0, leak_rate = 0.0, state_ok_rate = 1.0 | `rerun_2026-10-16/oai_concurrency_stress_result.json` |
| Numeric-hallucination suppression (runtime numeric-provenance checking) | 2,000 scenarios (1,000 grounded + 1,000 hallucinated) | detection_rate = 1.0 (0 miss), false_intercept_rate = 0.0, 7,174 numbers checked | `rerun_2026-10-16/hallucination_rerun_2000.json` (audit_sha256 26b01aa0556d7a02) |

Honest notes: scenarios are parameterized synthetic flows (seed 20261016), not production LLM traffic;
the driver (`rerun_2026-10-16/rerun_hallucination_2000.py`) calls the verifier interface only —
interface field names are neutralized (metric_a..h) and the real schema is a closed-source asset.
Driver self-correction is recorded in the artifact: v1 driver contained a year token ("2026年"), which
the checker correctly rejected as ungrounded (333 cases) — that is interface-contract behavior, not a
false positive; scenarios were fixed and rerun. Per-file SHA-256 in `SHA256SUMS_rerun.txt`.
