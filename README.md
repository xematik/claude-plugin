# Xematik diagram review

This plugin lets Claude Code review diagrams with you in the Xematik editor.
The agent draws a diagram. You comment on it in the editor. The agent reads your feedback and updates the diagram.

This repository is a read-only mirror. Changes come from the main project.

## Commands

- `/xematik-diagram-review:review-plan` draws a plan as a diagram and opens a review.
- `/xematik-diagram-review:check-drift` compares a diagram with the code and opens a review of the differences.

## Plan review hook

The plugin has an opt-in hook for the `ExitPlanMode` tool.
Set `XEMATIK_PLAN_REVIEW=1` in the environment of Claude Code to turn it on.
The agent must then open a diagram review of the plan first.

## Install

1. Add the marketplace: `/plugin marketplace add xematik/claude-plugin`
2. Install the plugin: `/plugin install xematik-diagram-review@xematik`
3. Install the `xematik-channel` bridge (see below).

## Install the bridge

The bridge pushes your feedback into the running Claude Code session.
Without it, the agent polls for feedback.

1. Open https://github.com/xematik/claude-plugin/releases.
2. Download the file for your system. Download `SHA256SUMS` too.
3. Check the file.
   - Linux: `sha256sum -c SHA256SUMS --ignore-missing`
   - macOS: `shasum -a 256 -c SHA256SUMS --ignore-missing`
4. Rename the file to `xematik-channel`.
5. Run `chmod +x xematik-channel`.
6. Move the file to a directory on your `PATH`.

## Channels

Channels let your feedback wake the running Claude Code session.

1. Start Claude Code with the flag `--dangerously-load-development-channels plugin:xematik-diagram-review@xematik`.
   Use the same flag with `--resume`. Without it, feedback does not wake a resumed session.
2. The bridge keeps its login in `~/.config/xematik/channel.json`.
   Create the folder `~/.config/xematik` first.
   In a sandbox, allow the folder. For nono, use `--allow ~/.config/xematik`.
3. The login link shows in the result of `watch_review` and as a channel message. Open it in your browser.
4. On a `stopped` message, fix the reason. Then ask the agent to watch the review again.

## MCP server only

The MCP server works without the plugin:

```
claude mcp add --transport http xematik https://xematik.com/mcp
```

## License

MIT. See `LICENSE`.
