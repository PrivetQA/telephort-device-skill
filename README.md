# Telephort device skill

Instructions for any AI agent that uses the Telephort SDK to create a device the same way the create form does: internet, model, GPU, keybox, then an optional Play Store update or sign-in.

The instructions are `.agents/skills/telephort-device/SKILL.md`.

Create, update, and delete are SDK REST calls (`devices.create`, `devices.update`, `devices.delete`). They are not MCP tools.

Raw file: https://raw.githubusercontent.com/PrivetQA/telephort-device-skill/main/.agents/skills/telephort-device/SKILL.md

## Use with any agent

Point the agent at this repository, or give it the raw URL above. The SDK lives in the Telephort backend at `sdk/typescript` and `sdk/python`. The client needs a base URL, a team id, and an access token.

Agents that read `AGENTS.md` or the Agent Skills layout pick the file up when this repository is the project:

- `.agents/skills/telephort-device/SKILL.md` is the shared copy
- `.cursor/skills/telephort-device` and `.claude/skills/telephort-device` point at that same file
