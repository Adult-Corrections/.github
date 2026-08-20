# JBoss 8 / JDK 21 Migration Tasks

Use this migration in two explicit phases to avoid out-of-order manual edits.

1. **Task A (required first):** `JBoss8_Upgrade_Task_A_OpenRewrite.md`
2. **Task B (after Task A is complete):** `JBoss8_Upgrade_Task_B_PostRewrite.md`

**Rule:** Do not manually change `javax` imports before Task A preflight passes and `rewrite:run` has completed.
