# Godot Game Development Skills

**Portuguese-language skills for AI-assisted game development with Godot 4.**

This repository contains reusable skills designed to help AI agents work more effectively and safely with Godot projects.

The collection currently covers three complementary areas: game programming, UX/UI development and operational control when an AI agent has direct access to project files or Godot through MCP.

## Available Skills

| Skill | Version | Main focus |
| --- | --- | --- |
| [`godot-programacao-para-jogos`](godot-programacao-para-jogos/) | `4.0.0` | Architecture, GDScript, game systems, saves, debugging and testing |
| [`godot-ux-ui-para-jogos`](godot-ux-ui-para-jogos/) | `4.0.0` | UX, UI systems, accessibility, layouts, navigation and visual validation |
| [`godot-mcp-controle-validacao`](godot-mcp-controle-validacao/) | `1.0.0` | MCP control, authorization, validation, regression prevention and operational safety |

## Which Skill Should I Use?

### Godot — Programação para Jogos

Use `godot-programacao-para-jogos` when the task involves:

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

Use `godot-ux-ui-para-jogos` when the task involves:

- interface design;
- UX flows;
- menus and navigation;
- responsive layouts;
- focus and input;
- accessibility;
- procedural interface content;
- clarity and usability;
- visual validation.

The skill is intended to connect UX/UI decisions with the actual technical systems of a Godot project.

### Godot MCP — Controle, Segurança e Validação

Use `godot-mcp-controle-validacao` when an AI agent can directly inspect, modify or execute a Godot project through tools such as Codex and Godot MCP.

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

Its rules were developed from a progressive benchmark of 27 Godot MCP tests performed during development of Golem's Mandate.

## Quick Start

Each skill is distributed as a complete folder.

To use one:

1. Choose the skill that matches your task.
2. Download or clone this repository.
3. Copy the **entire skill folder** into the skills directory supported by your AI agent or development environment.
4. Keep `SKILL.md`, `agents/`, `assets/` and `references/` together.
5. Start a new task with the skill available to the agent.

Do not copy only `SKILL.md`.

The supporting files contain metadata, references and additional instructions that are part of the skill.

## Using the Skills Together

The three skills are designed to complement one another.

A typical combination is:

**Programming**

`godot-programacao-para-jogos`

Handles architecture, GDScript, gameplay systems, saves, implementation and testing.

**UX/UI**

`godot-ux-ui-para-jogos`

Handles interface design, accessibility, layouts, navigation and user-experience validation.

**Operational control**

`godot-mcp-controle-validacao`

Adds authorization boundaries, MCP execution rules, independent validation and safer direct project access.

For tasks that involve direct AI modification of a Godot project, the MCP skill can be used together with either of the other two.

## Skill Structure

A typical skill directory contains:

```text
skill-name/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── assets/
│   └── icon.svg
└── references/
