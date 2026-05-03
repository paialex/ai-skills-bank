---
name: project-tutorial-builder
description: Build reusable, interactive onboarding tutorials from a live software repository. Use when the user asks to create a project tutorial, ramp-up guide, practical learning flow, developer onboarding walkthrough, tutorial prompt, tutorial context file, or project-specific tutorial skill based on README files, docs, source code, tests, configs, CLI/API surfaces, examples, or repository behavior.
---

# Project Tutorial Builder

## Workflow

Create a tutorial package from the current repository state. Do not hardcode project names, domain examples, or stale assumptions. Infer the tutorial from live files and clearly mark gaps that need user input.

1. Inspect the repository before drafting:
   - Prefer `rg --files`, then read README files, docs, manifests, config samples, tests, examples, and likely entrypoints.
   - Identify commands from package/build files, scripts, CLI definitions, Makefiles, Docker files, test fixtures, docs, and `--help` output when safe.
   - Treat tests, examples, fixtures, and docs as evidence of real user workflows.
2. Prepare `tutorial-context.md` using `references/analyze-project-for-tutorial.md`.
3. Prepare `interactive-tutorial-prompt.md` using `references/generate-interactive-tutorial.md`.
4. Create a project-specific tutorial skill only when repeat use is likely, using `references/create-project-tutorial-skill.md`.

## Output Rules

- Keep the generated tutorial interactive and practical: present a short project explanation, then a menu of learning paths.
- Offer developer-friendly choices such as fastest path, full tutorial, first example, configuration, local validation, debugging, and advanced features.
- Prefer copy-paste snippets and small exercises over passive documentation.
- Do not automatically execute project commands while tutoring unless the user explicitly requests execution. Provide commands and ask the user to paste output when feedback is needed.
- Explain inferred facts with evidence. If a feature, command, or workflow is ambiguous, ask a focused question or label it as uncertain.
- Keep artifacts generic and project-derived. Do not include unrelated sample product names or domain-specific assumptions.

## Artifact Defaults

Place generated tutorial artifacts in a discoverable project location:

- Prefer `docs/tutorial-context.md` and `docs/interactive-tutorial-prompt.md` when `docs/` exists.
- Otherwise prefer `tutorial-context.md` and `interactive-tutorial-prompt.md` at the project root.
- For a generated project-specific skill, use `project-tutorial-skill/` unless the repository already has a skills/plugins convention.

## Quality Bar

Before finishing, verify that:

- The tutorial context cites the files or commands used as evidence.
- The interactive prompt can be reused after the project evolves by re-running analysis.
- The tutorial does not claim command behavior that was not found in docs, source, tests, or help output.
- Missing setup, credentials, services, or environment assumptions are called out explicitly.
