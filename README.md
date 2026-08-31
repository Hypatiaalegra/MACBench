# MACBench

**A benchmark for sustained, multi-turn frontend web development.**

MACBench measures whether a coding agent can keep a real project coherent across a
long conversation. Each of the 28 instances is a *whole repository built from
scratch* over 16–36 sequential user turns, where later turns depend on decisions
made much earlier. Requirements are revealed progressively — features grow, get
revised, and pick up delayed constraints — so an agent cannot succeed by
one-shotting a single prompt.

The headline finding: agents reliably *survive* these sessions but rarely *satisfy*
them. Models keep the codebase buildable to the end, yet accumulate silent
regressions that leave the final artifact incomplete.

Paper: 

## What makes it hard

| | |
|---|---|
| **Long horizon** | 700 user turns across 28 repositories (25.0 avg, up to 36) |
| **Real interdependence** | 2,425 cross-turn dependency edges (86.6 per repository) — turn *n* routinely relies on turn *n−k* |
| **Executable evaluation** | 946 weighted checklist items scored against a *running* application, not against diffs |
| **No leakage** | Failures return the observed symptom (build error, broken behavior), never the expected fix |

## Dataset

`data/macbench.jsonl` — one JSON object per repository, 28 lines.

```jsonc
{
  "qid": "task_102",
  "description": "...",           // one-line goal of the finished app
  "round_num": 26,
  "stack_profile": {              // null where the stack is open: the agent picks
    "type": "spa_react", "backend_required": false,
    "persistence": "localstorage", "realtime": "none"
  },
  "test_mode": "http",
  "contexts": [                   // the user turns, in order
    { "round_id": 0, "prompt": "...", "tags": ["New Requirement"], "scenario_tags": [] }
  ],
  "checklist": [                  // graded against the FINAL artifact
    { "checklist_id": 0, "description": "...", "weight": 2, "judge_mode": "playwright" }
  ],
  "dependency": [                 // which turns are load-bearing, and for whom
    { "round_id": 0, "critical_check": {
        "description": "...", "depended_by_rounds": [1, 2, 15], "verify_mode": "playwright" } }
  ],
  "verifier_specs": [             // test code for judge_mode == "verifier" items
    { "verifier_case_id": "...", "test_code": "import { describe, it, expect } ..." }
  ]
}
```

18 repositories pin a framework (9 React, 9 Vue); the remaining 10 leave
`stack_profile` null and let the agent choose. Checklist weights encode functional
importance: **3** critical (59.1%), **2** important (34.8%), **1** minor (6.1%).

## Evaluation

Two complementary judges, routed per checklist item by `judge_mode`:

- **`playwright`** (713 items, 75.4%) — a browser agent drives the running app,
  navigating pages and triggering interactions to verify user-facing behavior.
- **`verifier`** (233 items, 24.6%) — Vitest/Pytest test code from `verifier_specs`
  (415 cases) for requirements with executable oracles.

During the session, dependency-critical turns trigger a lightweight **viability
check**. On failure the agent receives only the *symptom* and gets a bounded number
of repair attempts, which keeps long sessions executable without revealing hidden
behavior.

### Metrics

For repository *i*, let `V_i` be the weighted checklist score and `r_i` the number
of repair attempts used.

| Metric | Definition |
|---|---|
| **CSR** | passed checklist items / all checklist items (unweighted) |
| **Avg.** | `mean(V_i)` |
| **Adj. Avg.** | `mean(V_i · γ^r_i)`, γ = 0.9 — penalises reliance on repairs |
| **BSR** | fraction ending on a buildable artifact |
| **MCR** | fraction completing every main turn |
| **ISR** | fraction completing every turn *and* passing every checklist item |
| **FPSR** | ISR, restricted to runs that never needed a repair |



## Citation

<!-- TODO: add BibTeX on publication. -->

## License

Released under the [MIT License](LICENSE).
