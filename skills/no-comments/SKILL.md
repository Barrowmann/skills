---
name: no-comments
description: "Review a scoped change for removable comments and suppressions, then fix the code that makes them necessary."
disable-model-invocation: true
license: MIT
metadata:
  source: "https://github.com/backnotprop/pstack"
  revision: "c81f1bf680803f9e3f0cade6adb62593f43fb61f"
  local-adaptation: "Standalone review with one writer and verified constraint handling"
---

# No comments

Use the caller's files or diff. Otherwise review added or changed comments in the working diff from the merge base with the branch's base, defaulting to `main`. Include staged and unstaged changes and new source files. If a base is unavailable, use local changes against `HEAD` and state the scope. Read surrounding code for evidence and keep edits within the requested scope.

1. Give a read-only subagent the resolved scope and [Comment Sicko review guide](references/comment-sicko.md). Use the harness's supported delegation tool when available and permitted. Otherwise run the guide yourself as a review pass before editing.
2. Verify each finding against the scoped diff and source. Trace the relevant symbol or decision history when a comment claims a constraint. Check scoped lint and type suppressions the review missed. If the review is incomplete, recheck the affected findings once and proceed with the findings you can verify. For review-only requests, finish with a report here.
3. Remove redundant comments. For an accepted `MUST KILL`, fix the named code at its root within scope and remove the comment or suppression when the fix makes it redundant. Sketch the smallest design that addresses those findings if a structural change is needed. Keep unresolved findings open when the necessary change is outside scope. Remove a correctness or safety suppression together with its fix and run the affected check.
4. If a comment records a live constraint, offer the cheapest enforcement that fits the code, such as a type, runtime check, test, or lint rule. Apply it when the caller approves it. Keep the comment while it is the only record of a live constraint, and remove it once the constraint is recorded elsewhere, enforced, or shown to be obsolete.
5. Check the final diff. Report the number of comments removed, code fixes and checks, `REVIEW` keeps, and any open `MUST KILL` findings or proposed enforcement.
