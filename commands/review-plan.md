---
description: Review the current plan as a diagram with the user in the Xematik editor, then update the plan from their feedback.
---

# Review the plan as a diagram

Follow the `diagram-review` skill. Call `get_agent_guide` first.

1. Take the plan you are working on. Read the real code it touches.
2. Draw the plan as a component diagram at `module` level.
   - Add `@code <alias> -> <path>[::symbol]` anchors for parts that exist.
   - Use `(new)` as the path for parts that do not exist yet.
3. Call `open_review` with the diagram text, the `level`, and intent `plan`. Give the returned url to the user.
4. Wait for feedback in this order:
   - `watch_review`, if the tool is available.
   - `xematik-channel wait <session_id> --after 1` with the Bash tool and `run_in_background`.
   - Polling with `await_feedback`.
   Then end your turn and tell the user you wait for their feedback in the browser.
5. When feedback arrives, update the plan to match it. Do not edit code. If the diagram needs a change, call `push_revision`.
6. Call `close_review` when the user is done. Then show the updated plan and ask to exit plan mode.
