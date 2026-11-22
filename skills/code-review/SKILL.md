---
name: code-review
description: "Review changes since a fixed point (commit, branch, tag, or merge-base) across three independent axes: Standards, Spec, and Defensive Gates (normal-path reliability). Runs available reviews in parallel sub-agents and reports each axis separately. Use when reviewing a branch, a PR, work-in-progress changes, or changes since a given ref."
---

Three-axis review of the diff between `HEAD` and a fixed point the user supplies:

- **Standards**: does the code conform to this repo's documented coding standards?
- **Spec**: does the code faithfully implement the originating issue / spec?
- **Defensive Gates**: do defensive checks, readiness gates, and recovery mechanisms contain failures without unnecessarily blocking valid operations?

Run the available axes as **independent parallel sub-agents** against the same pinned diff and baseline, then aggregate their findings. Give each agent source evidence rather than conclusions from another axis.

## Process

### 1. Pin the fixed point

Whatever the user said is the fixed point (a commit SHA, branch name, tag, `main`, `HEAD~5`, etc.). If they didn't specify one, ask for it.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the list of commits via `git log <fixed-point>..HEAD --oneline`.

Before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty. A bad ref or empty diff should fail here, not inside the review sub-agents.

### 2. Identify the spec source

Look for the originating spec, in this order:

1. Issue references in the commit messages (`#123`, `Closes #45`, GitLab `!67`, etc.). Fetch them using the project's tracker instructions. If no tracker is configured, suggest `/setup-matt-pocock-skills` and continue to the next spec source.
2. A path the user passed as an argument.
3. A spec file under `docs/`, `specs/`, or `.scratch/` matching the branch name or feature.
4. If nothing is found, ask the user where the spec is. If they say there isn't one, the **Spec** sub-agent will skip and report "no spec available".

### 3. Identify the standards sources

Search the repository for files that document its coding standards. Include `CODING_STANDARDS.md` and `CONTRIBUTING.md` whenever present.

On top of whatever the repo documents, the Standards axis always carries the **smell baseline** below: a fixed set of Fowler code smells (_Refactoring_, ch.3) that applies even when a repo documents nothing. Two rules bind it:

- **The repo overrides.** A documented repo standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Each smell is a labelled heuristic ("possible Feature Envy"), never a hard violation. Like any standard here, skip anything tooling already enforces.

Each smell reads *what it is* → *how to fix*; match it against the diff:

- **Mysterious Name**: a function, variable, or type whose name doesn't reveal what it does or holds. → rename it; if no honest name comes, the design's murky.
- **Duplicated Code**: the same logic shape appears in more than one hunk or file in the change. → extract the shared shape, call it from both.
- **Feature Envy**: a method that reaches into another object's data more than its own. → move the method onto the data it envies.
- **Data Clumps**: the same few fields or params keep travelling together (a type wanting to be born). → bundle them into one type, pass that.
- **Primitive Obsession**: a primitive or string standing in for a domain concept that deserves its own type. → give the concept its own small type.
- **Repeated Switches**: the same `switch`/`if`-cascade on the same type recurs across the change. → replace with polymorphism, or one map both sites share.
- **Shotgun Surgery**: one logical change forces scattered edits across many files in the diff. → gather what changes together into one module.
- **Divergent Change**: one file or module is edited for several unrelated reasons. → split so each module changes for one reason.
- **Speculative Generality**: abstraction, parameters, or hooks added for needs the spec doesn't have. → delete it; inline back until a real need shows.
- **Message Chains**: long `a.b().c().d()` navigation the caller shouldn't depend on. → hide the walk behind one method on the first object.
- **Middle Man**: a class or function that mostly just delegates onward. → cut it, call the real target direct.
- **Refused Bequest**: a subclass or implementer that ignores or overrides most of what it inherits. → drop the inheritance, use composition.

### 4. Load the defensive-gate review skill

Read the installed `$defensive-gate-review` `SKILL.md` using its path from the skills catalog. Its necessity, scope, failure-semantics, net-reliability, and evidence rules are the source of truth for the **Defensive Gates** axis; keep those rules in that skill rather than duplicating them here.

Scope this axis to mechanisms introduced or materially affected by the diff, following the necessary callers and downstream paths. Label unrelated pre-existing problems as context rather than findings against this change.

If the skill is unavailable, report **Defensive Gates: unavailable** and continue the other available axes. A missing spec only skips the Spec axis.

### 5. Spawn the available sub-agents in parallel

Run the available sub-agent calls together in the foreground, each in a fresh context window. Wait for their reports before step 6.

**Standards sub-agent prompt** should include:

- The full diff command and commit list.
- The list of standards-source files you found in step 3, **plus the smell baseline from step 3** pasted in full (the sub-agent has no other access to it).
- The brief: "Report, per file/hunk where relevant, (a) every place the diff violates a documented standard: cite the standard (file + the rule); and (b) any baseline smell you spot: name it and quote the hunk. Distinguish hard violations from judgement calls: documented-standard breaches can be hard, but baseline smells are always judgement calls, and a documented repo standard overrides the baseline. Skip anything tooling enforces. Under 400 words."

**Spec sub-agent prompt** should include:

- The diff command and commit list.
- The path or fetched contents of the spec.
- The brief: "Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that wasn't asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong. Quote the spec line for each finding. Under 400 words."

**Defensive Gates sub-agent prompt** should include:

- The same pinned diff command and commit list.
- The resolved path to the installed `defensive-gate-review/SKILL.md`, with an instruction to read and apply it.
- The relevant standards and spec sources already collected; identify any missing sources explicitly.
- The brief: "Apply defensive-gate-review to this diff and the call paths needed to establish its effects. Review mechanisms introduced or materially affected by these changes. Follow the skill's causal-evidence, minimal-adjustment, and validation requirements. When a gate is explicitly required by the spec, distinguish implementation compliance from the reliability cost of that requirement. Report that cost as a contract/design concern where appropriate, preserving the concrete invariant. Keep substantiated findings separate from hypotheses and state any material coverage limits."

If the spec is missing, skip only the Spec sub-agent and report "no spec available". If the defensive-gate skill is unavailable, skip only that axis and report "skill unavailable".

### 6. Aggregate

Present the reports under `## Standards`, `## Spec`, and `## Defensive Gates`, verbatim or lightly cleaned. Keep each axis's findings and conclusions separate; cross-reference shared root causes without collapsing the axes or reranking findings across them. Mark skipped or unavailable axes explicitly rather than treating them as zero findings or a pass.

End with a one-line summary: finding counts and the worst issue within each completed axis, with hypotheses and unavailable axes identified separately. Do not select a single winner across axes.

## Why three axes

A change can satisfy one axis while raising findings in another:

- Code that follows every standard but implements the wrong thing → **Standards pass, Spec fail.**
- Code that does exactly what the issue asked but breaks the project's conventions → **Spec pass, Standards fail.**

- Code that follows the standards and faithfully implements a spec may still add an unnecessary global gate or make auxiliary failures block valid operations → **Standards pass, Spec pass, Defensive Gates finding.**

Report the axes independently so implementation compliance does not hide a reliability cost, and reliability concerns do not silently rewrite the agreed spec.
