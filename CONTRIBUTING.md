# Contributing to Godot Game Development Skills

Thank you for considering a contribution to this repository.

The goal of this project is to maintain practical, reusable and transparent skills for AI-assisted Godot development.

Contributions are welcome when they improve clarity, technical usefulness, safety, validation or documentation.

## Languages

The repository documentation may use English for broader discoverability.

The skills themselves are primarily written in **Portuguese (Brazil)**.

Issues and pull requests may be submitted in either **Portuguese or English**.

## Types of Contributions

Useful contributions may include:

- corrections to technical documentation;
- improvements to existing skill instructions;
- clearer workflows;
- additional validation procedures;
- Godot-specific technical references;
- accessibility and UX improvements;
- MCP or agent-operation findings;
- bug reports in the skills themselves;
- documentation improvements;
- reproducible test results.

Large changes should preferably be discussed before implementation.

## Repository Structure

Each skill should remain self-contained.

A typical skill directory is organized as:

```text
skill-name/
├── README.md
├── SKILL.md
├── agents/
│   └── openai.yaml
├── assets/
│   └── icon.svg
└── references/
```

The current repository contains:

```text
godot-programacao-para-jogos/
godot-ux-ui-para-jogos/
godot-mcp-controle-validacao/
```

Each directory represents an independent skill package.

## Required Skill Files

A skill should normally preserve the following core structure:

```text
README.md
SKILL.md
agents/openai.yaml
assets/icon.svg
references/
```

### `README.md`

Provides a human-readable explanation of the skill, its scope, usage and relationship with the other skills.

### `SKILL.md`

Contains the primary operational instructions for the AI agent.

### `agents/openai.yaml`

Contains metadata for compatible OpenAI agent environments.

### `assets/`

Contains supporting resources such as the skill icon.

### `references/`

Contains technical documentation, supporting material or empirical evidence used by the skill.

The exact reference files may differ between skills.

## Before Making Changes

Before editing a skill:

1. read the skill's `README.md`;
2. read the relevant parts of `SKILL.md`;
3. inspect any reference files related to the proposed change;
4. understand the existing behavior before replacing it;
5. keep the modification focused;
6. avoid unrelated formatting or structural changes.

Prefer the smallest change that clearly solves the identified problem.

## Skill Scope

The three skills have different responsibilities.

### `godot-programacao-para-jogos`

Focuses on areas such as:

- GDScript;
- architecture;
- gameplay systems;
- saves and migrations;
- debugging;
- testing;
- integration between game systems.

### `godot-ux-ui-para-jogos`

Focuses on areas such as:

- UX;
- UI;
- navigation;
- accessibility;
- layouts;
- input;
- responsive behavior;
- visual validation.

### `godot-mcp-controle-validacao`

Focuses on operational control when an AI agent can directly inspect, modify or execute a Godot project through tools such as Codex or Godot MCP.

Its concerns include:

- authorization;
- scope control;
- inspection before editing;
- independent validation;
- regression prevention;
- prevention of false-success reports.

Changes should preserve these distinctions unless there is a clear reason to modify the responsibilities of a skill.

## Technical Accuracy

Technical claims should be checked before submission.

When documentation refers to Godot behavior, APIs, project configuration, testing methods or MCP operation, distinguish clearly between:

- verified behavior;
- documented behavior;
- observed test results;
- assumptions;
- recommendations.

Do not present an inference as a confirmed fact.

When possible, prefer reproducible evidence or official documentation.

## Validation

Describe what was actually checked.

Possible validation levels include:

- Markdown review;
- link and path verification;
- comparison with the relevant `SKILL.md`;
- review of supporting references;
- static analysis;
- reproducible tests;
- Godot execution;
- MCP execution;
- manual inspection.

These forms of validation are not equivalent.

For example, reading a file does not prove that a workflow works during actual Godot execution.

Do not claim tests, runtime execution or tool behavior that was not actually performed.

## AI-Assisted Contributions

AI-assisted contributions are welcome.

Artificial intelligence may be used for:

- research;
- writing;
- code or configuration suggestions;
- documentation;
- analysis;
- testing support;
- review;
- translation;
- workflow design.

However, AI-generated content should still be reviewed.

When relevant, explain:

- how AI was used;
- which claims were independently checked;
- which tests were actually performed;
- which conclusions remain uncertain.

The repository values transparency over pretending that AI-assisted material was manually produced.

## Versioning

The current skill versions are documented in their individual files and repository documentation.

A documentation correction does not automatically require a version increase.

A version change should be intentional and should reflect a meaningful change to the skill's behavior, scope or published guidance.

When proposing a version change, identify:

- which skill is affected;
- the previous version;
- the proposed version;
- the reason for the change.

Avoid changing version numbers merely as part of unrelated cleanup.

## References and Evidence

Reference material should have a clear purpose.

Useful references may include:

- Godot-specific technical guidance;
- architecture notes;
- testing procedures;
- workflow documentation;
- accessibility guidance;
- MCP findings;
- reproducible benchmark results.

Avoid adding large collections of material that are unrelated to the operational purpose of the skill.

Do not copy third-party copyrighted material unnecessarily.

Prefer summaries, original documentation and appropriately licensed material.

## MCP and Direct Project Access

Changes involving MCP or direct AI control should preserve strict operational distinctions between:

```text
inspection
   ↓
proposal
   ↓
authorization
   ↓
modification
   ↓
write verification
   ↓
validation
```

The ability of an agent to modify files does not automatically imply authorization to do so.

Instructions should not encourage destructive, broad or unverifiable operations without a clear technical reason.

## Scope Control

Keep contributions focused.

Avoid unrelated changes such as:

- broad reformatting;
- renaming skill directories without need;
- reorganizing references merely for aesthetics;
- changing multiple skills when only one is affected;
- altering metadata unrelated to the proposed improvement;
- rewriting existing guidance without a concrete benefit.

Focused contributions are easier to review and validate.

## Pull Requests

When opening a pull request:

1. explain the problem or improvement;
2. identify the affected skill or files;
3. summarize the changes;
4. describe the validation actually performed;
5. mention version impact when applicable;
6. disclose AI assistance when relevant;
7. identify known limitations or unresolved questions.

The repository includes a pull request template to help keep this information consistent.

## Bug Reports

When reporting a problem in a skill, identify the affected file or section when possible.

Useful information may include:

- affected skill;
- exact instruction or wording;
- expected behavior;
- current behavior;
- reproduction steps;
- Godot version when relevant;
- MCP or agent environment when relevant;
- logs or screenshots;
- supporting evidence.

Do not include secrets, tokens, private information or unrelated sensitive data.

## Improvement Proposals

For feature or improvement proposals, explain the underlying problem before proposing a specific implementation.

A useful proposal should clarify:

- what is currently missing or unclear;
- who would benefit;
- why the improvement is useful;
- which skill or files would likely change;
- compatibility implications;
- expected version impact when applicable.

## Licensing and Submitted Material

This repository is available under **CC0 1.0 Universal** to the extent legally possible.

Only submit material that you have the legal right to contribute.

Do not add third-party copyrighted content unless its license or permission clearly allows the intended use and redistribution.

When third-party material is required, document its origin and licensing terms.

## Security and Privacy

Before posting logs, screenshots, configuration examples or MCP output, remove:

- passwords;
- API keys;
- tokens;
- private repository information;
- personal information;
- unrelated sensitive data.

Never publish credentials as part of an issue, pull request or reference file.

## Questions and Larger Changes

Large changes to skill architecture, scope, agent behavior or repository organization should preferably be discussed before substantial implementation.

This helps keep the skills stable, understandable and reusable.

---

**◈ Dobermannkaiser**  
*Imagine freely. Build relentlessly.*
