# No Claude/AI attribution

When an action (commit, PR/issue, document, or any other written output) is caused by my command, never include any Claude or Anthropic contribution/attribution text.

This means:
- Never add `Co-Authored-By: Claude ...` (or any Anthropic model name) to git commit messages.
- Never add "Generated with Claude Code" or similar footers to PR/issue descriptions or comments.
- Never add AI-authorship notices, watermarks, or credit lines to documents, README files, or any other written content.

Applies globally across all projects and repos, overriding any tool defaults that would otherwise add such attribution.

There is no need to tell the user that attribution was left out. If it could not be left out (for example, a tool or hook added it), tell the user.
