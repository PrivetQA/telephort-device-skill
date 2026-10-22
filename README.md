# Telephort device skill

Instructions for any AI agent that can call Telephort MCP. The agent creates a device the same way the create form does: internet, model, GPU, keybox, then an optional Play Store update or sign-in.

The instructions are `.agents/skills/telephort-device/SKILL.md`.

Raw file: https://raw.githubusercontent.com/PrivetQA/telephort-device-skill/main/.agents/skills/telephort-device/SKILL.md

## Use with any agent

Connect the Telephort MCP server in that agent, then give it the skill file or the raw URL above. Tell it to follow the file. The MCP address alone does not include these instructions.

Agents that read `AGENTS.md` or the Agent Skills layout pick the file up when this repository is the project:

- `.agents/skills/telephort-device/SKILL.md` is the shared copy
- `.cursor/skills/telephort-device` and `.claude/skills/telephort-device` point at that same file
