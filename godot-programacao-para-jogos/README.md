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
