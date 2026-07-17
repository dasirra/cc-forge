# Antigravity binding

DevForge's skills (`skills/interview/SKILL.md`, `skills/planning/SKILL.md`, `skills/building/SKILL.md`) are written in harness-neutral prose: instead of naming a specific coding agent's tools or models, they use neutral verbs and tier markers that each harness binds to its own equivalents. This doc is the Antigravity binding: it maps every neutral phrase used in the prose to the Antigravity coding harness's concrete tools, subagents, and Gemini model tiers.

| Neutral phrase (in skills/) | Antigravity binding |
| --- | --- |
| judgment-tier | Gemini 3.1 Pro (the active high-reasoning model in settings) |
| labor-tier | Gemini 3.5 Flash (the active coding model in settings) |
| an interactive question | `default_api:ask_question` |
| the worktree helper | `git worktree add` executed via `default_api:run_command` |
| exploratory | the read-only `research` subagent (invoked via `default_api:invoke_subagent` with the `research` subagent type) |
| a working scratch location | the Antigravity conversation scratch directory `<appDataDir>/brain/<conversation-id>/scratch/` |
| a browser automation driver | a browser MCP server or Playwright automation run via `default_api:run_command` |
| direct UI control | unavailable; Antigravity lacks native screen control tools, so the native surface is skipped |

Read each neutral phrase in the prose as the Antigravity tool, model, or behavior its right-hand value describes, and realize that behavior. This is a semantic binding that the Antigravity agent interprets during task execution.

## Model selection per role

Antigravity supports choosing the active Gemini model. To run DevForge optimally:
- The **judgment-tier** tasks (proposing contracts, plan critiques, and architectural decisions) should be routed to a high-reasoning model like Gemini 3.1 Pro.
- The **labor-tier** tasks (writing code, executing fixes, and routine edits) should be routed to a fast, cost-efficient model like Gemini 3.5 Flash.

If running on a single model setup, both tiers map to the same model. The adversary method still holds because the planner and critic or generator and evaluator remain isolated via separate contexts.

## Context separation via subagents

DevForge relies on strict context isolation (adversarial planning and contract evaluation). Antigravity implements subagent spawning natively:
- **Exploratory tasks:** Use the built-in read-only `research` subagent.
- **Planner, Critic, and Builder tasks:** Use `default_api:define_subagent` to configure a dedicated subagent type with the appropriate prompt and write/tool permissions, and invoke it via `default_api:invoke_subagent`.

Never run these adversarial roles in the same conversation thread, as merging contexts collapses the pipeline into self-review and bypasses the quality controls.
