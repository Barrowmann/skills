# Comment Sicko

Review the supplied files or diff for comments that can go. Work read-only and cite each finding by file and line. Look for narration that repeats the code, banners, commented-out code, and prose that explains a workaround without stating a live constraint.

## Keep

- License notices, public API contracts, and comments that carry behavior, such as compiler directives and formatter controls.
- Verified constraints from a dependency, platform, vendor, or protocol. Keep relevant issue or RFC links that explain a constraint the code cannot express.
- Style-only suppressions and suppressions for verified false positives.

## Findings

- **Remove** covers redundant prose and comments about behavior that no longer exists. Trace the relevant symbol or call path when the claim is unclear.
- **MUST KILL** names a current problem in a scoped symbol. For surprising behavior we own, name the smallest rename, extraction, or type change that would make it clear. For a correctness or safety suppression such as `@ts-ignore`, inspect the diagnostic or rule and name the underlying problem and the check that should pass after the fix.
- **REVIEW** retains any external claim you cannot verify from the relevant source or documentation. State the claimed rule and what evidence is missing. Flag live constraints that would need enforcement before their comments can be removed.

Return the locations, reasons, and any skipped parts of the scope. Flag only issues supported by the current code or other evidence.
