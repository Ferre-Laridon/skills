---
name: workflow-feature
description: End-to-end feature delivery — plan, branch, TDD cycles, review, docs commit, pre-merge, PR. Use when starting a new feature task or when #{N} is assigned.
---
trig: new-feature·new-task·#{N}-assigned
in: requirement·#{N}·branch-scope

## Sequence
1. §2.plan — scan lessons → write todo.md → candidates → chosen+gates
2. §3.branch — `#{N}-slug` from main
3. commit-workflow step 1 — plan commit
4. tdd-workflow cycles — red→green→refactor (repeat per behavior)
   - each cycle: test commit → feat commit → refactor commit
5. §5.review — min 2 cycles · ∥(reviewer + security-reviewer) · §4∀pass
6. commit-workflow step 5 — docs commit (final)
7. §5.pre-mrg — `git log #{N}`: verify sequence
8. pr-creation → label:T · size:{s-count}c → merge

## Gates
- §4∀pass at each commit · test-commit < impl-commit · ∀commit = 1 unit · §5.P before merge

## Abort
- §5.stuck@5cyc → handoff · revert.depth>2 → abandon+ADR · scope-creep → halt → replan

