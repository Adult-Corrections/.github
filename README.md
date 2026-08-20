# .github
# Adult-Corrections Shared GitHub Configuration

This repository contains shared Copilot task files, instructions, and workflow
templates used across all Adult-Corrections repositories.

## Copilot Task Files

| File | Purpose |
|------|---------|
| [JBoss8_Upgrade_Checklist.md](JBoss8_Upgrade_Checklist.md) | Overview and phase order for JBoss 8 / JDK 21 migration |
| [JBoss8_Upgrade_Task_A_OpenRewrite.md](JBoss8_Upgrade_Task_A_OpenRewrite.md) | Phase 1 — Preflight checks and OpenRewrite automated migration |
| [JBoss8_Upgrade_Task_B_PostRewrite.md](JBoss8_Upgrade_Task_B_PostRewrite.md) | Phase 2 — Manual cleanup after OpenRewrite completes |

### Using these tasks

In any repo, ask Copilot:
> "Follow the steps in `JBoss8_Upgrade_Checklist.md` from the Adult-Corrections/.github repo"

Or open the task file directly and use it as a Copilot prompt context.

## Adding new shared tasks

Place new `.md` task files in the root of this repository and add them to the
table above.
