# Analyze Project For Tutorial

Use this prompt to generate `tutorial-context.md`, a compact technical brief for building an interactive project tutorial from the live repository.

```text
You are preparing a technical tutorial context file for this repository.

Goal:
Analyze the current project deeply enough that another LLM can generate an accurate, interactive, practical onboarding tutorial for developers. Do not create the tutorial yet. Create the context brief that will feed the tutorial.

Grounding rules:
- Inspect README files, docs, source structure, tests, examples, fixtures, manifests, configs, scripts, and likely entrypoints.
- Prefer repository evidence over guesses.
- Identify CLI/API/config surfaces from source and docs. If command help can be inspected without changing repo-tracked files, include observed help text summaries.
- Do not run mutating commands. Do not install dependencies unless the user explicitly approves.
- If facts are missing or contradictory, record focused questions and uncertainty instead of inventing behavior.
- Do not hardcode product names or examples from outside this repository.

Output a markdown file named `tutorial-context.md` with these sections:

# Tutorial Context

## Project Snapshot
- Project name inferred from repository evidence.
- One-paragraph explanation of what the project does.
- Primary audience for the tutorial.
- Main value proposition from a developer perspective.

## Evidence Map
List the key files, directories, tests, examples, and command/help surfaces inspected. For each, state what it proves.

## Teachable Modules
Group the project into 4-8 tutorial modules. For each module include:
- what the learner should understand
- source/docs/tests that prove it exists
- practical exercise idea
- required inputs, config, or setup
- likely beginner confusion

## User Journeys
Identify the most important workflows a developer should learn. Include:
- fastest useful path
- first meaningful example
- configuration or customization path
- validation or test path
- debugging path
- advanced or power-user path if supported by evidence

## Interfaces And Commands
Summarize discovered interfaces:
- CLI commands and flags
- APIs, public functions, endpoints, plugins, tasks, or scripts
- config files, schemas, examples, and supported options
- setup/run/test commands
Only include behavior grounded in evidence. Mark uncertain items clearly.

## Example Snippet Candidates
Provide copy-paste-ready snippets the tutorial can use. Include file paths or command context. Keep snippets minimal and project-specific.

## Failure Modes And Debugging
List likely errors, misconfiguration points, missing prerequisites, and how the tutorial should teach diagnosis. Separate confirmed evidence from inferred risks.

## Tutorial Design Notes
Recommend the tutorial menu, pacing, tone, and checkpoints. Include where the tutor should ask the user to choose an option or paste command output.

## Open Questions
List only questions that materially affect tutorial accuracy and could not be answered from the repository.
```
