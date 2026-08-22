# Godot MCP — Controle, Segurança e Validação

**Skill em português para operação controlada de Codex, acesso direto a arquivos e Godot MCP em projetos Godot 4.x.**

**Versão atual:** `1.0.0`

Esta skill foi criada para governar como um agente de inteligência artificial deve operar um projeto Godot quando possui capacidade de inspecionar arquivos, modificar conteúdo ou executar o motor através de ferramentas como Codex e Godot MCP.

Ela funciona como uma camada de **controle operacional, autorização, validação e segurança**.

Não substitui uma skill de programação ou UX/UI.

## Quando usar

Use esta skill quando uma IA puder:

- acessar diretamente arquivos de um projeto Godot;
- editar GDScript;
- editar cenas;
- modificar configurações;
- utilizar Codex;
- utilizar Godot MCP;
- executar o projeto;
- coletar logs ou debug;
- investigar e corrigir bugs;
- alterar sistemas de save/load;
- criar testes;
- modificar código de produção;
- realizar operações que precisam de validação independente.

## Objetivo principal

O objetivo desta skill é reduzir problemas comuns em desenvolvimento assistido por IA, como:

- alterações realizadas sem autorização clara;
- mudanças fora do escopo solicitado;
- edição antes de compreender o projeto;
- falsos sucessos reportados por ferramentas;
- correções realizadas sem reprodução do problema;
- regressões não identificadas;
- confusão entre análise estática e execução real;
- promoção indevida de uma versão como estável;
- alterações destrutivas ou difíceis de reverter.

## Regra central

A regra operacional mais importante da skill é:

> **Inspecionar primeiro, explicar o escopo e obter autorização explícita antes de qualquer alteração.**

O fluxo básico é:

```text
inspecionar
      ↓
identificar o problema
      ↓
explicar a alteração proposta
      ↓
listar arquivos e escopo
      ↓
pedir autorização
      ↓
aguardar aprovação
      ↓
executar a alteração
      ↓
verificar a escrita
      ↓
validar o comportamento
      ↓
relatar resultados e limitações
