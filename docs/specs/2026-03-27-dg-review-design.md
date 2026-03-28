# DG Review Skill — Design Spec

**Date:** 2026-03-27
**Status:** Approved

## Overview

An adversarial code review skill that dispatches two subagents — Gilfoyle (attacker) and Dinesh (defender) — who debate about the user's code in character. The back-and-forth produces better reviews because genuine adversarial tension surfaces issues a single reviewer would miss, and successful defenses validate code more strongly than a rubber stamp.

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Architecture | Two separate subagents (Option C) | One mind can't argue with itself. Real tension requires independent agents. |
| Debate structure | Sequential (Approach 1) | Gilfoyle attacks → Dinesh defends → repeat. Natural flow, clean convergence. |
| Round control | Adaptive with cap | Agent detects convergence. Default cap of 5. User can override or extend. |
| Trigger | Explicit invocation only (`/dg`) | Keep it simple. User decides when to unleash. |
| Scope | Git diff + arbitrary files | Default to git diff. Accept file/path as argument. |
| Tone | Banter sandwich (Option B) | In-character debate, followed by structured actionable summary. |
| Command | `/dg` | Short, both names represented, Gilfoyle-approved efficiency. |

## Architecture

```
User invokes /dg [target] [rounds]
        │
        ▼
   Orchestrator (main agent)
   ├── Parses args
   ├── Gathers code context
   ├── Runs debate loop:
   │   ├── Dispatch Gilfoyle agent → critique
   │   ├── Check convergence
   │   ├── Dispatch Dinesh agent → defense
   │   ├── Check convergence
   │   └── Repeat until converged or cap hit
   └── Synthesizes final review
```

## Agent Design

**Gilfoyle:** Deadpan, technically precise, finds real issues, scales venom to offense severity. Outputs BANTER + structured FINDINGS.

**Dinesh:** Defensive but competent, concedes when wrong, defends when right, dismisses nitpicks. Outputs BANTER + structured FINDINGS with [concede]/[defend]/[dismiss] tags.

## Convergence

Debate ends when:
1. Gilfoyle raises no new issues
2. Dinesh concedes everything
3. Both are cycling on the same points
4. Round cap reached (user asked to stop)

## Output

Final summary categorizes issues by debate outcome:
- **Critical:** Gilfoyle won, Dinesh conceded immediately
- **Important:** Gilfoyle won after debate
- **Contested:** Dinesh defended successfully
- **Dismissed:** Both agreed it's a nitpick

Includes banter highlights, strengths, and a tongue-in-cheek score.

## Files

```
dg/
  SKILL.md              # Orchestrator instructions
  gilfoyle-agent.md     # Gilfoyle subagent prompt
  dinesh-agent.md       # Dinesh subagent prompt
```
