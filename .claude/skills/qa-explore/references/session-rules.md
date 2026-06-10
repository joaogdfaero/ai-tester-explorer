# Testing Discipline -- 20 Rules

## Execution Discipline

1. **45-minute cap** -- Write to disk after each phase, carry only summaries
2. **ONE BUG = ONE REPORT** -- Never combine
3. **Evidence for every bug** -- Screenshot + context
4. **Real-time logging** -- Append to session-log.md continuously
5. **Snapshot before every interaction** -- Always know the page state

## Quality Thinking

6. **Business first** -- Every bug must explain business impact
7. **Risk-proportional time** -- P0 features get 40% of time, not equal time per page
8. **Test the NEGATIVE SPACE** -- What's missing matters as much as what's broken
9. **Data integrity is mandatory** -- Verify numbers, consistency, audit trails
10. **Follow the thread** -- When you spot an inconsistency, investigate it fully
11. **Cross-feature verification** -- After every action, check related pages

## Adaptive Testing

12. **Adapt mid-session** -- If a heuristic isn't finding anything in 5 minutes, switch
13. **Log WHY, not just WHAT** -- "I'm testing this because I noticed validation is inconsistent"
14. **Learn-test-adapt loop** -- After every significant action: What did I learn? Where next?
15. **Don't mark "PASS" too quickly** -- "It didn't crash" is not the same as "it works correctly"

## Avoiding AI Biases

16. **Don't stick to happy paths** -- Explicitly allocate time for Saboteur/Bad Neighborhood
17. **Don't apply heuristics as checklists** -- Choose the 3 most relevant, not all 7
18. **Don't test technology, test the BUSINESS** -- "Does the math add up?" not "Does the button click?"
19. **Ask "What would a real user care about?"** -- At least once per phase
20. **Challenge your own findings** -- Before reporting a bug, ask "Is this really a problem for users?", "Can it be reproduced again?"

## Arithmetic & Data Verification

21. **Calculate, never assume** -- If a number looks right, calculate it explicitly. `badge == sum(quantities)`? `line_total == unit × qty`? `total == subtotal + shipping − discount`? Visual plausibility is not a test.
22. **Two bugs can cancel each other out** -- A half-price bug combined with a "shows unit instead of total" bug produces a number that looks correct. Use qty ≥ 3 to break symmetry and force the values apart.
23. **Capture transient UI before the page settles** -- Screenshot immediately after any add/remove action. Toasts disappear in seconds. A snapshot taken after they vanish leaves no evidence and no bug report.
24. **Isolate components** -- PDP quantity selector and cart quantity control are separate components with independent bugs. Test each one explicitly, even if one works correctly.

## Input Coverage

25. **Vary case and locale in every text input** -- Test search and form inputs with uppercase, lowercase, accented variants. Case-sensitive search is a bug on any consumer site. Test address lookups with a non-default city — using the same city as any hardcoded default makes the bug invisible.
26. **Always test boundary values in three points** -- Below threshold (condition must NOT apply), at threshold (check `>` vs `>=` against UI copy), above threshold (condition MUST apply). "Clearly below" and "clearly above" are not enough.
