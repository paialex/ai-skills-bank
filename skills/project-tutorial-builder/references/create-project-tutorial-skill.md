# Create Project Tutorial Skill

Use this prompt only when the project will benefit from a dedicated tutorial skill that can be reused by a team.

```text
You are creating a project-specific Codex skill for interactive developer tutorials.

Inputs:
- `tutorial-context.md`
- current README/docs/source/tests/config evidence
- existing repository conventions for skills/plugins, if any

Decision:
Create a dedicated skill only if at least one is true:
- The tutorial will be reused by multiple developers.
- The project has a non-trivial CLI/API/config surface.
- The tutorial needs stable workflow rules beyond a single prompt.
- The repo already contains skills, agents, plugins, or onboarding automation.

If a skill is not justified, explain why and keep `interactive-tutorial-prompt.md` as the primary artifact.

If creating the skill:
- Use a lowercase hyphenated skill name under 64 characters.
- Keep `SKILL.md` lean and repository-specific.
- Include frontmatter with only `name` and `description`.
- Put longer tutorial prompts, examples, or context in `references/`.
- Do not create README, installation guide, changelog, or extra docs inside the skill folder.

The generated skill must instruct Codex to:
- refresh or verify tutorial context from live repository files before teaching
- present a menu-driven tutorial
- avoid automatic command execution unless explicitly requested by the user
- teach with snippets, checkpoints, and practical exercises
- ask focused questions when repo evidence is missing

Recommended structure:

project-specific-tutorial/
├── SKILL.md
└── references/
    ├── tutorial-context.md
    └── interactive-tutorial-prompt.md

Validation:
- Check YAML frontmatter parses.
- Check the description clearly states when to use the skill.
- Check references are linked directly from `SKILL.md`.
- Check no hardcoded stale claims remain without evidence.
```
