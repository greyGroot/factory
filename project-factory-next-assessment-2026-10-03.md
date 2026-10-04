# Project Factory Next compared with your Factory blueprint

Assessment date: 3 October 2026. Reviewed repository: https://github.com/koldovsky/project-factory-next, `main` at `f13b896d26d57421f807e152a854f4392dd923df`.

Baseline: `agentic_software_factory_mvp_blueprint_ua.md` in this workspace, particularly sections 2, 5, 25, and 26.

## Recommendation

This repository is strongly aligned with your deterministic-verification principles. It supplies a useful local developer/evaluator kernel and a separate experiment assessor. It does not implement your two-agent, MCP-callable MVP.

Use it as a reference and, if the license permits your intended use, as a component behind your Factory interface. Keep QA Red, worker selection, workspace provisioning, MCP job lifecycle, selective skills, Langfuse, and experiment execution in your surrounding runtime. Your blueprint explicitly prefers evolving the existing lang-factory infrastructure; this review does not establish that replacing it would be cheaper or better.

The repository declares PolyForm Noncommercial 1.0.0. Its README explicitly says company/client use requires a separate commercial license. Resolve that before adopting its code for such work. A technical match does not resolve licensing. Sources: [README](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/README.md), [LICENSE](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/LICENSE).

## Strict comparison with the 21 MVP Definition of Done bullets

These classifications describe source coverage, not independently executed acceptance results. “Partial” means a useful related mechanism exists but the complete requirement is absent. Equal counting does not measure implementation effort.

| # | Blueprint requirement | Coverage | Evidence or gap |
|---|---|---|---|
| 1 | Separate factory project runs locally | Covered | Node CLI, local state store, executable demos; successful source CI. |
| 2 | Codex/Antigravity-compatible MCP client starts a run | Missing | No factory MCP server or tool definitions. |
| 3 | MCP returns run_id without waiting for the entire run | Missing | CLI awaits runFactory; no asynchronous service boundary. |
| 4 | Status/result/cancel | Partial | inspect and recover exist; no complete start/status/result/cancel job API. |
| 5 | Real QA Red worker | Missing | Control bundle is prepared externally; only developer adapters run. |
| 6 | QA Red produces contract/tests | Missing | init scaffolds a failing placeholder; automated QA contract authoring is absent. |
| 7 | QA Red technically cannot alter application source | Missing | No QA Red role/environment exists. |
| 8 | Real developer with independently configurable adapter/model | Partial | Codex/Claude developer and model selection exist; there is no separate verifier configuration. |
| 9 | Developer technically cannot alter verification artifacts | Partial | Separate directories, policy digests and runtime restrictions help; no complete infrastructure boundary. |
| 10 | Deterministic PASS/FAIL controls repair loop | Covered | Controller evaluates exact case sets, exit/report consistency and integrity. |
| 11 | Strict max attempts | Covered | Task budget and bounded worker/evaluator execution. |
| 12 | Isolated workspace/worktree | Partial | Disjoint directories enforced; caller must provision candidate checkout/worktree and isolation. |
| 13 | Selective/progressive skill discovery | Missing | Profiles provide instructions; no skill-index/select/load pipeline. |
| 14 | Selected skill IDs and hashes recorded | Missing | Policy/runtime/artifact hashes exist, but no selected-skill ledger. |
| 15 | Langfuse trace with node/gate/worker metadata | Missing | Local JSON evidence/history exists; no Langfuse integration. |
| 16 | Only trustworthy runtime usage is recorded | Partial | Raw worker output is captured; no normalized usage reporting. |
| 17 | User feedback linked to run | Missing | Experiment rows accept humanMinutes/quality; no met_intent/rating/rework feedback capture per run. |
| 18 | One-factor A/B tool rejects additional differences | Partial | Strict paired-data assessor exists; no execution tool or complete single-factor diff enforcement. |
| 19 | A/B starts from same base commit in different worktrees | Missing | No paired workspace provisioning/execution. |
| 20 | Integration example demonstrates complete flow | Partial | Real good/bad developer/evaluator and browser demos; no QA Red → developer → gate → MCP example. |
| 21 | QA Green stays external | Covered | Independent integration/release decisions remain external; no hidden final certification agent. |

Count: **4 covered, 7 partial, 10 missing**. This indicates limited exact-MVP coverage despite strong conceptual alignment. Several missing requirements belong to one surrounding integration layer, so this is not a prediction of remaining engineering time.

Primary implementation sources: [runner](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/src/runner.mjs), [adapters](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/src/adapters.mjs), [CLI](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/bin/factory.mjs), [store](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/src/store.mjs), [learning](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/src/learning.mjs), [experiment schema](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/schemas/experiment.schema.json).

## Particularly useful mechanisms

- Reviewed, sealed control bundles with a separately supplied expected policy digest.
- Exact evaluator case identities, structured reports, process exit consistency and artifact validation.
- Bounded repair attempts and subprocess deadlines/output limits.
- Build preparation before candidate evaluation, preventing failed rebuilds from accepting stale exports.
- Candidate, evaluator and evidence content digests; evidence inspection checks for subsequent modification.
- Real visual acceptance across routes/viewports/states, including geometry, pixel, behavior, overflow and accessibility checks.
- Paired experiment validation with disjoint splits, frozen arm metadata, conservative uncertainty and explicit inconclusive results.

Source: [architecture](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/docs/architecture.md), [self-improvement](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/docs/self-improvement.md).

## Using your subscriptions

### Codex

The existing adapter invokes native Codex CLI through `codex exec`, passes the prompt on stdin, requests JSON events, supports explicit model selection and uses workspace-write sandboxing. It disables workspace shell network access and login shells. Prepare dependencies before dispatching this worker.

Official documentation states that local Codex supports ChatGPT subscription login and API-key login, and that `codex exec` reuses saved CLI authentication. For your local subscription workflow, authenticate with `codex login`, inspect `codex login status`, and run the factory with `--agent codex`. This path still consumes account limits; it is not a separate unlimited allowance. API-key authentication uses Platform billing instead.

Sources: [authentication](https://learn.chatgpt.com/docs/auth), [non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode), [repository runtime guide](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/docs/agent-runtimes.md).

The adapter expects a native executable and rejects .cmd/.bat/.ps1 wrappers. On Windows, supply the trusted native executable path when needed. Its fixture tests and successful CI do not establish live compatibility with your account/model. A disposable authenticated smoke task is still required.

### Antigravity

Two useful arrangements are possible:

1. **Available without adding an adapter:** use Antigravity for task clarification, architecture and reviewed acceptance authoring; run this factory with Codex as developer; have Antigravity inspect the resulting evidence and perform external QA Green. Antigravity can also edit a candidate externally, followed by `--agent manual` evaluation. Manual mode evaluates the supplied candidate once rather than automatically repairing it.
2. **Integration to implement:** add an Antigravity adapter using the official `agy` CLI. It has native Windows support, account sign-in, headless execution, model selection, stdin streaming and JSON output. Bind a configured worker to the candidate workspace, enforce timeouts and scoped permissions, and parse terminal status as well as process exit. Then expose separate verifier/developer configurations.

Example shape for a future adapter, not a factory command implemented today:

```text
agy --input-format stream-json --output-format stream-json --model <available-model-slug> --sandbox
```

Send a JSON `user` event on stdin. The adapter should use the signed-in Google account for the intended subscription path. Google AI Pro/Ultra offer higher Antigravity rate limits and credit pools. API-key mode is a distinct Gemini API path; it is not evidence that your personal subscription pays arbitrary API requests.

Sources: [installation/authentication](https://antigravity.google/docs/cli/install/), [headless mode](https://antigravity.google/docs/cli/headless/), [pricing](https://antigravity.google/pricing).

Antigravity headless can exit zero while soft-denying a permission-dependent tool. Therefore process success must remain separate from candidate acceptance. Configure scoped allowed actions rather than treating unrestricted execution as the integration solution. The current repository validates adapter names as manual/codex/claude, so substituting agy with `--executable` does not create an Antigravity adapter: it would still receive the other adapter's arguments.

### Usage and experiments

Both current official CLI guides document structured token usage: Codex emits turn.completed usage; Antigravity emits result.usage. Parse and persist those fields when actually present. In a persistent Antigravity streaming session, usage/duration counters are cumulative, so record deltas or use fresh sessions to prevent double counting.

Keep token usage, subscription quota and monetary cost separate. The experiment assessor requires numeric costUSD. That assumption needs an explicit measurement protocol or a reviewed schema/assessor change for subscription comparisons where attributable per-run dollars are unavailable. Do not fabricate API-equivalent cost or use zero to mean unknown. Wall-clock time, retries, deterministic outcomes and user rework are useful initial metrics.

## Suggested integration order

1. Resolve reuse licensing; compare this kernel with existing lang-factory before choosing a migration.
2. Run one representative local Codex subscription task, with meaningful acceptance and a deliberately failing candidate. Preserve the evidence.
3. Add QA Red and enforce its writable scope; seal its contract before developer execution. Enforce developer/evaluator separation and provision worktrees.
4. Add the async MCP start/status/result/cancel boundary around the existing run lifecycle.
5. Add the native Antigravity adapter and independent role/model configuration.
6. Add selective skills, Langfuse, user feedback, then a one-factor paired execution harness from the same baseline.

A suitable first comparison is the same reviewed task, same QA contract and base commit, with only developer runtime changed between Codex and Antigravity. Treat this as a runtime-package comparison, not proof that model capability alone caused a difference.

## Verification limits

I inspected source, schemas, tests and documentation through GitHub; I did not install the project, execute its tests locally, or run paid workers. The reviewed commit's [Factory CI run](https://github.com/koldovsky/project-factory-next/actions/runs/35723180795) completed successfully. The repository's [validation record](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/docs/validation.md) reports 102 tests with no failures/skips and explicitly says no paid model calls were made.

Hashes and disjoint folders detect drift but do not establish full hostile-code isolation. The repository documents that separately constrained builder, candidate, evaluator and release environments are an external integration: [trust boundaries](https://github.com/koldovsky/project-factory-next/blob/f13b896d26d57421f807e152a854f4392dd923df/docs/trust-and-deployment.md).
