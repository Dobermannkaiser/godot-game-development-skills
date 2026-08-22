# Godot — Programação para Jogos

**Skill em português para programação e engenharia de jogos com Godot 4.x.**

**Versão atual:** `4.0.0`

Esta skill foi criada para orientar agentes de inteligência artificial em tarefas de programação, arquitetura, depuração, testes e manutenção de projetos desenvolvidos com Godot.

Ela busca tornar o desenvolvimento assistido por IA mais controlado, verificável e seguro, especialmente quando o agente possui acesso direto aos arquivos do projeto ou ao Godot através de ferramentas como MCP.

## Quando usar

Use esta skill em tarefas relacionadas a:

- GDScript;
- arquitetura de projetos Godot;
- cenas, nós, Resources e sinais;
- sistemas de gameplay;
- economia e sistemas de gerenciamento;
- narrativa e relacionamentos;
- saves e migrações;
- depuração;
- erros de parser ou runtime;
- testes;
- regressões;
- versionamento;
- integração entre sistemas;
- preparação e revisão de entregas;
- Codex e Godot MCP.

## Princípios principais

A skill prioriza alguns princípios importantes durante o desenvolvimento.

### Preservar uma base estável

A última versão explicitamente testada e aprovada deve ser tratada como a base estável do projeto.

Alterações novas não devem substituir silenciosamente essa base.

### Inspecionar antes de modificar

Antes de editar um projeto, o agente deve compreender sua estrutura, versão do Godot, cena principal, autoloads, sistemas envolvidos, saves e testes existentes.

### Respeitar o escopo autorizado

Permissão para alterar uma parte do projeto não significa autorização para modificar outras áreas.

Mudanças de escopo devem ser identificadas antes da implementação.

### Separar implementação de validação

Uma alteração escrita em disco não é considerada correta apenas porque uma ferramenta informou sucesso.

Sempre que possível, a mudança deve ser verificada de forma independente.

### Diferenciar análise estática de execução real

Análise textual, parsers auxiliares e inspeção de arquivos não substituem a execução do projeto no Godot.

Quando o motor não tiver sido executado, essa limitação deve ser declarada.

## Fluxo geral

O fluxo recomendado pela skill segue aproximadamente esta sequência:

```text
entender o pedido
        ↓
inspecionar o projeto
        ↓
identificar escopo e dependências
        ↓
reproduzir ou diagnosticar
        ↓
propor a mudança
        ↓
obter autorização quando necessária
        ↓
implementar
        ↓
validar
        ↓
relatar resultado e limitações
```

O objetivo é evitar alterações prematuras e reduzir o risco de regressões ou modificações fora do escopo.

## Estrutura da skill

```text
godot-programacao-para-jogos/
├── README.md
├── SKILL.md
├── agents/
│   └── openai.yaml
├── assets/
│   └── icon.svg
└── references/
```

### `README.md`

Apresenta uma visão geral da skill para leitura humana, incluindo seu propósito, uso e relação com as demais skills do repositório.

### `SKILL.md`

Contém as instruções principais, regras e fluxo operacional da skill.

### `agents/openai.yaml`

Contém os metadados utilizados por ambientes compatíveis de agentes OpenAI.

### `assets/`

Contém recursos auxiliares da skill, incluindo seu ícone.

### `references/`

Contém documentação complementar sobre diferentes áreas do desenvolvimento com Godot.

Entre os temas cobertos estão:

- arquitetura e GDScript;
- persistência e versionamento;
- testes e diagnóstico;
- sistemas de jogo;
- integração com UI;
- entrega e checklists;
- operação com Codex e Godot MCP.

## Como usar

Copie ou disponibilize a pasta completa:

```text
godot-programacao-para-jogos/
```

no ambiente de skills utilizado pelo seu agente.

Não copie apenas o `SKILL.md`, pois a skill utiliza também os materiais presentes em `references/`, além dos metadados e recursos complementares.

A localização exata da pasta de skills depende do agente ou ambiente utilizado. Consulte a documentação desse ambiente para saber onde disponibilizar a pasta completa.

Depois, utilize a skill em tarefas de desenvolvimento Godot compatíveis com seu escopo.

Exemplos de tarefas:

> Revise a arquitetura deste projeto Godot antes de propor alterações.

> Investigue este erro de GDScript e determine a causa antes de corrigir.

> Analise o sistema de save e verifique riscos de incompatibilidade.

> Implemente esta mecânica preservando a versão estável existente.

## Validação

A skill diferencia diferentes níveis de evidência.

Uma análise pode envolver:

- inspeção de arquivos;
- revisão de arquitetura;
- busca de referências;
- análise estática;
- execução de testes;
- execução real do projeto no Godot;
- validação manual pelo usuário.

Esses níveis não devem ser tratados como equivalentes.

Por exemplo, verificar que um script parece sintaticamente correto não comprova, por si só, que o comportamento funciona durante a execução do jogo.

Quando uma forma de validação não puder ser realizada, essa limitação deve ser informada.

## Uso com Codex e Godot MCP

Quando o agente possui acesso direto ao projeto por Codex, MCP ou ferramentas equivalentes, a skill recomenda separar claramente:

```text
inspeção
   ↓
proposta
   ↓
autorização
   ↓
alteração
   ↓
verificação da escrita
   ↓
validação
```

A capacidade técnica de modificar arquivos não deve ser interpretada como autorização automática para fazê-lo.

Para regras específicas de controle operacional, autorização e validação de operações MCP, utilize também a skill complementar:

[`godot-mcp-controle-validacao`](../godot-mcp-controle-validacao/)

## Integração com UX/UI

Quando uma alteração técnica envolve interface, navegação, acessibilidade, layouts ou comportamento visual, esta skill pode ser utilizada em conjunto com:

[`godot-ux-ui-para-jogos`](../godot-ux-ui-para-jogos/)

A skill de programação continua responsável pela arquitetura e integração técnica, enquanto a skill de UX/UI adiciona orientação específica sobre experiência do jogador e validação da interface.

## Skills complementares

### [`godot-ux-ui-para-jogos`](../godot-ux-ui-para-jogos/)

Para tarefas de UX, UI, acessibilidade, layouts, navegação e validação visual.

### [`godot-mcp-controle-validacao`](../godot-mcp-controle-validacao/)

Para controle de autorização, operação direta através de MCP, validação independente e segurança operacional.

## Idioma

A skill é escrita principalmente em **português do Brasil**.

Pode responder em outro idioma quando solicitado pelo usuário.

## Licença

Esta skill faz parte do repositório **Godot Game Development Skills** e está disponibilizada sob os termos da **CC0 1.0 Universal**, nos limites dos direitos que o responsável pelo repositório pode legalmente renunciar.

Consulte o arquivo [`../LICENSE`](../LICENSE) para o texto jurídico completo.

---

**◈ Dobermannkaiser**  
*Imagine freely. Build relentlessly.*
