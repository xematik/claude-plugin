---
description: Check that the code anchors in the repo's diagrams still match the code, and offer an update review for stale ones.
---

# Check diagram drift

Follow the `diagram-review` skill. This command works offline for the check itself.

1. Run `xematik-channel check` with the Bash tool. It reads the diagrams under `docs/`.
   - For another folder, run `xematik-channel check <dir>`.
   - Exit 0 means every anchor still matches. Tell the user and stop.
   - Exit 1 means stale anchors. Each output line is `file:line alias -> path[::symbol]: reason`.
   - Exit 2 means a usage error. Show the message.
2. Show the stale anchors to the user as a short list, grouped by diagram file.
3. For each diagram file with stale anchors, read the real code. Fix the anchors in a copy of the diagram text. Keep the rest of the diagram as it is.
4. Offer the user an update review. If they agree, call `get_agent_guide`, then `open_review` with the fixed diagram, and give the returned url to the user.
5. When the review is closed, save the final diagram over the old file under `docs/`.
