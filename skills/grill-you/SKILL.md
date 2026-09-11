---
name: grill-you
description: Stress-test a plan through Expert, Critic, and Codebase Researcher agents instead of interviewing you.
disable-model-invocation: true
---

# Grill You

Grill the agents relentlessly until the plan survives evidence and counterexamples. Adapt `grilling`'s design tree and frontier rounds, but delegate technical decisions as well as factual research. This is a standalone workflow: use it instead of the user-answer loop in `grilling`.

The main agent is **Griller + Synthesizer**. Three independent subagents are the panel; the Synthesizer is not a fourth subagent. Use real subagent dispatch, not three personas in one response. Keep the exercise read-only: implementation, deployments, purchases, and other external actions require separate authorization. An invocation authorizes technical recommendations, not invented user preferences or operational consent.

## 1. Frame the tree

Read the supplied proposal and scope relevant repository instructions and entry points before dispatching. Capture the goal, success criteria, explicit constraints, non-goals, and authority already granted. If no topic can be recovered from the conversation or supplied material, ask only for the topic.

Maintain a compact decision ledger in working context:

`ID | question | prerequisites | status | answer / rationale | evidence | strongest objection / disposition | next check`

Statuses: `open`, `researching`, `settled`, `blocked`. Label claims as **fact** (source-backed), **inference** (reasoned), or **assumption** (unverified). An assumption is not a settled prerequisite when its falsity would change the decision.

The **frontier** is every open decision whose prerequisites are settled. Put dependent questions in later rounds; pending research holds only its descendants. Name newly discovered branches rather than silently dropping them. Completion of framing means every known decision and missing fact has an ID and explicit dependencies.

## 2. Dispatch the frontier

### Select models before agents

Build the panel from the models the user has already selected in roles or agent configuration, not from the provider's entire catalog. Resolve the Griller's **active** model from session/runtime metadata; the configured default may be different. Inspect the host's effective model mappings, agent configuration, and availability/spawn restrictions. Read only routing fields, not credentials. First identify the runtime and its exposed capabilities; role names, configuration paths, and dispatch APIs are runtime-specific, not portable assumptions.

- **Strength:** choose the strongest suitable configured model for each panel role, including Researcher. Follow an explicit user capability ranking first; otherwise prefer mappings the user designated for deep reasoning and known task-relevant capability over speed/cost defaults. Role names are intent signals, not benchmarks: label an inferred ranking and explain the choice; do not claim an objective winner without evidence. Break comparable choices using the user's configured preference order, not price. Respect required tools and runtime agent-type restrictions; report when these prevent the preferred model from being routed.
- **Critic diversity:** first exclude candidates using the Griller's underlying model, then choose the strongest remaining suitable candidate. Expert and Researcher may share the Griller's model. A different agent name, role alias, thinking level, provider proxy, or endpoint does not establish a different underlying model. Resolve known aliases; among comparably capable remaining models, prefer a different model family. If identity is unknown, report diversity as unverified rather than assuming it.
- **Routing:** distinguish panel roles from dispatch agent types. Select an allowed, suitable agent and supported model-selection route for the chosen candidate; do not hard-code panel roles to particular agent names. Resolve aliases and runtime fallback precedence before dispatch. A prompt asking a worker to use another model cannot change its execution model. Use invocation-local model/effort controls only if the exposed API supports them. Do not silently edit shared role mappings, agent definitions, or provider configuration to obtain a different model.
- **Selection gate:** record `panel role | dispatch agent | configured selector | resolved model | strength rationale | diversity status` before the first round. If the desired model has no permitted route, distinguish the best routable choice from the preferred one. If no distinct Critic is available or its identity cannot be verified, continue independent fact gathering but hold panel completion; ask for a suitable existing-model mapping or explicit permission to use a same-model/unverified Critic. Never describe that fallback as cross-model review.
- **Runtime check:** compare the selection with the actual model recorded by task results or runtime metadata, including retry fallbacks and revived agents. An agent's self-reported model is not proof. A Griller model change also requires a fresh diversity check. If routing changes invalidate the selection, replace the affected panel member and repeat the affected critique before settling its decisions; if runtime identity is unavailable, keep the limitation explicit under the selection gate.

### Apply the host's model controls

Use the same strength, diversity, and verification rules through the host's supported interface:

| Host capability | Selection route |
| --- | --- |
| **Explicit per-spawn model selection** | Pass the selected, user-configured model through the documented field, plus a compatible agent type if required. Verify the actual selection; do not invent parameter names or infer support from another harness. |
| **Agent/role mappings only** | Choose a suitable existing agent whose effective mapping reaches the selected model. Model roles without a dispatch route are preferences, not executable choices. |
| **Inherited model only** | Expert and Researcher can inherit it, but a distinct Critic is unavailable. Apply the selection gate; renaming a persona does not provide another model. |
| **Opaque or unavailable model metadata** | Mark strength/identity unverified and apply the selection gate. Report which capability is missing; do not guess a model from generated text. |

**OMP adapter only:** when running under OMP, inspect `modelRoles`, `task.agentModelOverrides`, and discovered agent frontmatter. Resolution is override, then agent model list, then parent fallback, expanding aliases; fallback order is not a strength ranking. `@slow` can signal deep-reasoning intent, but use its current mapping. Ordinary `task` and `eval.agent()` have no per-call model selector; use a suitable configured agent and respect mandatory `scout` research when required. For details read `omp://task-agent-discovery.md` and `omp://tools/task.md`. Other hosts use their own installed documentation and exposed schema, not OMP paths or aliases. The actual tool schema wins over examples in this skill.

### Run the round

For each round, number the frontier questions and state the Griller's provisional recommendation and what could falsify it. Dispatch the three roles concurrently through the host's native subagent facility: one batch when available, otherwise independent parallel spawns. Each gets the same immutable brief: goal, constraints, ledger, frontier IDs, relevant paths, known evidence, and this response contract:

`ID | answer or challenge | evidence + fact/inference/assumption labels | tradeoff / failure case | what would change this answer | follow-up dependencies`

When the dispatch tool supports an output schema, override specialist defaults with `{ "type": "object", "properties": { "report": { "type": "string" } }, "required": ["report"], "additionalProperties": false }`; `report` contains the contract above. This keeps a code-review verdict schema from masquerading as a design decision.

Provide all needed context explicitly; subagents do not inherit the conversation. Require read-only work, no nested delegation, and no formatters, linters, builds, or test suites. Use the selected agent/model route; rely on a dispatch default only when its resolved selection matches the intended one. Preserve configured reasoning effort unless a supported control and the user's policy justify changing it.

| Role | Assignment | Completion criterion |
| --- | --- | --- |
| **Expert** | Propose the smallest defensible solution for each decision. Compare the best alternative, name costs and assumptions, and answer on the user's behalf within the brief. | Every frontier ID has a recommendation or a precise missing prerequisite. |
| **Skeptic / Critic** | Independently attack the proposal and provisional recommendations. Seek concrete counterexamples, violated constraints, failure modes, and a stronger alternative. Distinguish blocking objections from acceptable tradeoffs; explain how each objection can be resolved. | Every frontier ID has its strongest material objection, or an explicit explanation of why no blocker was found. |
| **Codebase Researcher** | Verify factual prerequisites against code, callers, tests, configuration, and documentation. Cite paths with lines or symbols; use symbol-aware navigation when available. Separate current behavior from intended behavior. For absence claims, report search scope; unavailable evidence stays unknown. | Every requested fact is supported, contradicted, or unknown with the lookup performed. For a non-code topic, say repository evidence is not applicable and identify any relevant accessible sources. |

Keep Expert and Critic independent until their first answers arrive. While research runs, handle unrelated ready decisions. If the runtime lacks subagents, report the limitation rather than claiming a panel ran. A failed role is missing evidence, not assent: retry a transient failure or report the affected decisions blocked after available alternatives are exhausted.

## 3. Cross-examine and synthesize

Compare answers per ID; check decisive citations against their source before accepting them. User constraints outrank preferences; verified behavior outranks unsupported assertions about the codebase. Agent agreement is not evidence and majority vote does not resolve a contradiction.

Send the Expert's actual proposal and the Researcher's findings to the Critic for a targeted rebuttal wherever the initial critique did not cover the proposed choice. Send material objections back to the Expert to defend or revise the answer. Reuse live agents where possible; otherwise dispatch a replacement with the complete brief. Parallelize independent follow-ups; serialize only answers that require another answer. The Synthesizer owns the decision, not a vote.

Settle an ID only when:

- the answer satisfies the explicit constraints and its prerequisites are settled;
- decisive factual claims have evidence, with remaining uncertainty bounded;
- the strongest material objection has a recorded disposition: refuted by evidence, mitigated in the recommendation, or accepted as a tradeoff within the delegated authority;
- there is a concrete verification criterion for the proposed behavior.

Preserve dissent and the reason for rejecting an alternative. If new evidence invalidates a settled answer, reopen it and its dependent decisions. Recompute the frontier after synthesis and continue automatically; do not wait for the user to answer ordinary technical questions.

Emit a compact round record, not full agent transcripts:

```text
Round N — Q IDs
Question + provisional recommendation:
Expert:
Critic:
Researcher: evidence / unknowns
Synthesis: decision, rationale, objection disposition, verification criterion
Next frontier / blocked prerequisites:
```

A disagreement becomes a sharper question or a targeted factual lookup, not another repetition of positions. If two consecutive rounds on the same ID add no evidence, alternative, or objection disposition, mark it blocked with the exact missing fact or authority. Do not manufacture consensus to finish.

## 4. Close or escalate

Finish reachable branches before escalating. Ask the user only for a preference, authority, or inaccessible information that materially changes the answer and cannot be recovered from available sources. Batch those questions, give a recommended choice and tradeoff, and state which IDs each unblocks. Keep downstream decisions blocked; unresolved does not mean settled.

The design is **complete** only when every discovered in-scope branch is settled, the frontier is empty, no research is pending, and no blocking objection remains. An empty frontier with blocked nodes is **blocked**, not complete. Resume the tree when missing input arrives.

Deliver:

1. **Verdict:** complete or blocked; the recommended design in a short paragraph.
2. **Decision ledger:** settled answers, rationale, evidence, rejected alternatives / dissent, and verification criteria.
3. **Unresolved:** remaining assumptions, risks, and any batched user-only questions with affected IDs.
4. **Next action:** the concrete implementation or experiment to perform after approval.

Completion delivers a stress-tested recommendation. Stop before implementation and request approval unless the user has already explicitly authorized that implementation scope. Agent consensus never expands that scope.
