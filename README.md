# Godot Game Development Skills

**Portuguese-language skills for AI-assisted game development with Godot 4.**

This repository contains reusable skills designed to help AI agents work more effectively, transparently and safely with Godot projects.

The collection currently covers three complementary areas: game programming, UX/UI development and operational control when an AI agent has direct access to project files or Godot through MCP.

## Available Skills

| Skill | Version | Main focus |
| --- | --- | --- |
| [`godot-programacao-para-jogos`](godot-programacao-para-jogos/) | `4.0.0` | Architecture, GDScript, game systems, saves, debugging and testing |
| [`godot-ux-ui-para-jogos`](godot-ux-ui-para-jogos/) | `4.0.0` | UX, UI systems, accessibility, layouts, navigation and visual validation |
| [`godot-mcp-controle-validacao`](godot-mcp-controle-validacao/) | `1.0.0` | MCP control, authorization, validation, regression prevention and operational safety |

## Which Skill Should I Use?

### Godot — Programação para Jogos

Use [`godot-programacao-para-jogos`](godot-programacao-para-jogos/) when the task involves:

- GDScript;
- game architecture;
- gameplay systems;
- saves and migrations;
- debugging;
- automated or manual testing;
- economy and management systems;
- narrative systems;
- relationships and progression;
- integration between project systems.

It also includes practices for preserving stable project states, validating modifications and using the real Godot environment when runtime confirmation is necessary.

### Godot — UX e UI para Jogos

Use [`godot-ux-ui-para-jogos`](godot-ux-ui-para-jogos/) when the task involves:

- interface design;
- UX flows;
- menus and navigation;
- responsive layouts;
- focus and input;
- accessibility;
- localization;
- clarity and usability;
- visual validation.

The skill connects UX/UI decisions with the actual technical systems of a Godot project rather than treating interface design as an isolated visual layer.

### Godot MCP — Controle, Segurança e Validação

Use [`godot-mcp-controle-validacao`](godot-mcp-controle-validacao/) when an AI agent can directly inspect, modify or execute a Godot project through tools such as Codex and Godot MCP.

It defines practices for:

- explicit authorization before modifications;
- inspection before editing;
- independent validation after writes;
- scope control;
- bug reproduction before correction;
- regression testing;
- save/load and round-trip validation;
- prevention of false-success reports;
- safer use of destructive or version-control operations.

This skill acts as an additional operational layer. It does not replace the programming or UX/UI skills.

Its rules were developed and refined from a progressive benchmark of 27 Godot MCP tests performed during development of Golem's Mandate.

## Quick Start

Each skill is distributed as a complete folder.

To use one:

1. Choose the skill that matches your task.
2. Download or clone this repository.
3. Locate the complete directory for the chosen skill.
4. Copy or make that **entire skill directory** available to the skills system supported by your AI agent or development environment.
5. Keep `SKILL.md`, `agents/`, `assets/` and `references/` together.
6. Start a new task with the skill available to the agent.

For example, to use the programming skill, provide the complete directory:

```text
godot-programacao-para-jogos/
```

rather than copying only:

```text
godot-programacao-para-jogos/SKILL.md
```

The supporting files contain metadata, references and additional instructions that are part of the skill.

The exact installation or skills-directory location depends on the AI agent or environment being used. Follow that environment's documentation for where complete skill directories should be placed.

## Using the Skills Together

The three skills are designed to complement one another.

A typical combination is:

### Programming

```text
godot-programacao-para-jogos
```

Handles architecture, GDScript, gameplay systems, saves, implementation and testing.

### UX/UI

```text
godot-ux-ui-para-jogos
```

Handles interface design, accessibility, layouts, navigation and user-experience validation.

### Operational control

```text
godot-mcp-controle-validacao
```

Adds authorization boundaries, MCP execution rules, independent validation and safer direct project access.

For tasks that involve direct AI modification of a Godot project, the MCP skill can be used together with either of the other two.

For a task involving both gameplay code and interface changes, the programming and UX/UI skills can also be used together.

## Skill Structure

A typical skill directory contains:

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

### `README.md`

Provides a human-readable overview of the skill, its purpose, usage and relationship with the other skills.

### `SKILL.md`

Contains the primary operational instructions used by the AI agent.

### `agents/openai.yaml`

Contains integration metadata for compatible OpenAI agent environments.

### `assets/`

Contains supporting resources such as the skill icon.

### `references/`

Contains detailed technical documentation and supplementary material used by the skill.

Not every skill contains the same reference files, because each one covers a different area of Godot development.

## Design Principles

Although each skill has a different specialization, the collection follows common principles:

- inspect before modifying;
- preserve known stable states;
- keep authorization boundaries explicit;
- distinguish verified facts from assumptions;
- avoid unnecessary changes outside the requested scope;
- separate static analysis from actual runtime validation;
- verify important writes independently;
- avoid claiming tests that were not performed;
- prefer reproducible workflows over vague recommendations;
- describe limitations transparently.

## AI-Assisted Development

These skills are specifically designed for workflows where artificial intelligence participates directly in software and game development.

AI assistance may include:

- code generation;
- architecture analysis;
- debugging;
- documentation;
- testing support;
- interface analysis;
- project inspection;
- MCP-assisted operation.

The purpose of the collection is not to pretend that AI-assisted work was manually produced.

The goal is to make AI-assisted development more useful, controlled, testable and transparent.

## Contributing

Contributions that improve technical accuracy, clarity, validation, safety or documentation are welcome.

Before proposing a change, please read:

[`CONTRIBUTING.md`](CONTRIBUTING.md)

The repository also includes templates for bug reports, improvement proposals and pull requests.

## Language

The skills themselves are written primarily in **Portuguese (Brazil)**.

The repository-level documentation uses English where useful for broader discoverability.

Issues and contributions may be submitted in Portuguese or English.

## License

This repository is available under **CC0 1.0 Universal** to the extent legally possible.

You may copy, modify, distribute and reuse the material, including commercially, without requesting permission or being required to provide attribution, to the extent that the repository owner has rights that can legally be waived.

See [`LICENSE`](LICENSE) for the complete legal text.

Third-party material, if any, remains subject to the rights and licenses of its respective owners.

---

**◈ Dobermannkaiser**  
*Imagine freely. Build relentlessly.*
