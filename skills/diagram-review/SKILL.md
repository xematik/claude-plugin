---
name: diagram-review
description: Review and edit a diagram of the code with the user in the Xematik editor. Use when the user asks to "review a diagram with me", "edit a diagram with me", "diagram review", or "show me the architecture to edit".
---

# Diagram review

You and the user review one diagram together. The user edits it in the browser. You change the code.

## Steps

1. Call `get_agent_guide` first. It is the source of truth for the loop and the anchor syntax.
2. Read the real code. Build the diagram from it, not from memory.
3. Choose the level:
   - `system` for services or applications.
   - `module` for packages or folders.
   - `function` for a call flow.
4. Write the diagram text. Put anchors inside the body:
   - `' @level module` (use `%%` instead of `'` in Mermaid).
   - `' @code <alias> -> <path>[::symbol]` for each element. Use `(new)` as the path for code that does not exist yet.
5. Call `open_review` with the text and the `level`.
   - Pass the project modules as `palette_blocks`, a JSON array: `[{"name":"billing","path":"src/billing/"}]`.
   - The user drops these blocks on the diagram. Each one carries its code path.
6. Give the returned url to the user.
7. If the `watch_review` tool is available, call it with `session_id` and `after_revision` 1. Then end your turn and tell the user you wait for their feedback in the browser. Channel messages `<channel source="xematik-channel">` wake you. Act by the `kind`:
   - `feedback`: call `await_feedback` with the `after_revision` named in the text. Then act as in step 8.
   - `closed`: stop.
   - `login`: show the URL to the user.
   - `stopped`: tell the user the reason. Call `watch_review` again only if the user asks.
8. If `watch_review` is not available, or the user reports that no channel message came (channels are not active in this session), run `xematik-channel wait <session_id> --after 1` with the Bash tool and `run_in_background`. Then end your turn and tell the user you wait for their feedback. Claude Code wakes you when the command exits:
   - exit 0: feedback arrived. The output names the `await_feedback` call. Make it, then act by the intent as listed under polling.
   - exit 2: the review is closed. Stop.
   - exit 3: no feedback for 30 minutes. Ask the user if they still want the review.
   - exit 1: an error (for example not logged in). Read the output and use polling.
   After you push or reply, start the waiter again with `--after` set to your last handled revision.
9. Polling is the last choice. Use it only when `watch_review` and `xematik-channel wait` are not available. Loop on `await_feedback` with `session_id` and `after_revision`.
   - On status `waiting`, call it again at once. The user may take minutes. This is normal. Do not stop or ask the user.
   - On status `closed`, stop.
   - On status `feedback`, act by the intent:
     - `discuss` or `plan`: answer with `reply`. Do not edit code.
     - `apply`: edit the code, then call `push_revision` with the new diagram text.
10. Call `close_review` when the user is done. Then save the final diagram text under `docs/` (or the folder the user names), for example `docs/<name>.puml`. `xematik-channel check` reads that folder later and reports anchors that no longer match the code (`/xematik-diagram-review:check-drift`).

## Rules

- Never delete code without asking the user first.
- Treat every `@code` anchor as the link between an element and the code. Keep the anchors up to date in each pushed revision.

## Plan mode

- The command `/xematik-diagram-review:review-plan` reviews the current plan as a diagram with intent `plan`. Use `(new)` as the `@code` path for parts that do not exist yet.
- If `XEMATIK_PLAN_REVIEW=1` is set, the plugin refuses the first `ExitPlanMode` of each plan. Run `/xematik-diagram-review:review-plan`, then exit plan mode.
