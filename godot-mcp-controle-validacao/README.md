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
```

A existência de uma ferramenta capaz de modificar o projeto não significa que a modificação esteja automaticamente autorizada.

## Níveis operacionais

A skill organiza ações em quatro níveis de impacto.

### Nível 0 — Somente leitura

Inclui atividades como:

- ler scripts e cenas;
- procurar referências;
- localizar call sites;
- analisar configurações;
- comparar conteúdo;
- analisar logs já existentes;
- consultar informações do projeto;
- consultar a versão do Godot.

Essas operações não modificam o projeto.

Quando fazem parte da análise solicitada, normalmente podem ser realizadas sem autorização de escrita.

### Nível 1 — Execução do Godot

Inclui atividades como:

- iniciar o editor;
- executar o projeto;
- coletar output;
- realizar debug;
- interromper a execução.

A execução pode gerar caches ou alterar dados em `user://`, dependendo do projeto.

Se o usuário pediu explicitamente para executar ou testar o projeto, essa autorização vale dentro do escopo solicitado.

Quando a tarefa é apenas análise, execução pesada ou potencialmente mutável deve ser tratada separadamente.

### Nível 2 — Escrita no projeto

Inclui atividades como:

- editar `.gd`;
- editar `.tscn`;
- criar cenas;
- adicionar nodes;
- criar arquivos de teste;
- modificar `project.godot`;
- salvar alterações persistentes no projeto.

Essas operações exigem autorização explícita prévia dentro de um escopo definido.

### Nível 3 — Operações destrutivas ou de versionamento

Inclui operações como:

- apagar arquivos;
- mover ou renomear arquivos;
- substituir estruturas importantes;
- executar limpeza massiva;
- restaurar ou resetar conteúdo;
- alterar schema de saves;
- realizar migrações;
- remover assets;
- reestruturar pastas;
- promover uma versão como nova base estável.

Essas ações exigem autorização específica e um escopo ainda mais claro.

## Autorização por escopo

Autorização não é transitiva.

Por exemplo, autorização para modificar:

```text
scripts/player.gd
```

não significa autorização automática para modificar:

```text
scenes/player.tscn
project.godot
tests/
```

Se outro arquivo de produção se tornar necessário durante a tarefa, o agente deve:

```text
parar
  ↓
preservar a evidência
  ↓
explicar a nova necessidade
  ↓
pedir ampliação do escopo
  ↓
aguardar autorização
```

Isso reduz alterações laterais e torna o processo mais auditável.

## Inspeção antes de edição

Antes de pedir autorização para modificar um projeto, a skill recomenda identificar, quando relevante:

- versão do projeto;
- versão do Godot;
- `project.godot`;
- cena principal;
- scripts envolvidos;
- cenas envolvidas;
- autoloads;
- call sites;
- testes existentes;
- saves e schemas relacionados;
- diferença entre código de produção e harness de teste.

O agente deve trabalhar a partir das APIs, arquivos e estruturas encontradas no projeto real.

Não deve inventar interfaces ou dependências que não foram encontradas.

## Validação independente

Uma ferramenta informar:

```text
success
```

não é evidência suficiente de que a alteração realmente persistiu ou funciona corretamente.

Um retorno de sucesso pode significar apenas que a chamada terminou sem erro aparente.

Depois de uma escrita relevante, a skill recomenda buscar evidência independente.

Dependendo da alteração, isso pode envolver:

- reler o arquivo;
- comparar o conteúdo persistido;
- verificar a cena salva;
- executar o projeto;
- observar output real do Godot;
- executar testes;
- realizar teste de regressão;
- validar save/load;
- solicitar teste humano quando apropriado.

## Diferenciar tipos de validação

A skill distingue claramente:

```text
análise estática
       ≠
arquivo escrito
       ≠
execução MCP
       ≠
execução real do Godot
       ≠
teste humano
```

Esses níveis fornecem tipos diferentes de evidência.

Por isso, o agente não deve declarar:

> validado no Godot

se o projeto não foi realmente executado.

Da mesma forma, não deve declarar:

> validado pelo usuário

se o usuário não confirmou o resultado.

## Bugs e correções

Antes de corrigir um bug, a skill prioriza:

```text
evidência
   ↓
reprodução quando possível
   ↓
diagnóstico
   ↓
causa provável
   ↓
solução mínima
   ↓
autorização
   ↓
correção
   ↓
validação
   ↓
regressão
```

O objetivo é evitar alterações baseadas apenas em suposições.

Quando um problema não puder ser reproduzido, essa limitação deve ser informada.

## Base estável

Uma versão não deve ser promovida automaticamente a base estável apenas porque passou por verificações técnicas.

A skill diferencia:

```text
tecnicamente validado
```

de:

```text
aprovado como nova base estável
```

Uma IA pode identificar uma versão como candidata tecnicamente validada.

A promoção para base estável oficial depende de aprovação explícita do usuário responsável pelo projeto.

## Benchmark de 27 testes

As regras desta skill foram desenvolvidas e refinadas a partir de um benchmark progressivo de **27 testes com Godot MCP** realizado durante o desenvolvimento de **Golem's Mandate**.

O benchmark investigou comportamentos relacionados a:

- leitura;
- escrita;
- persistência;
- criação e alteração de cenas;
- propriedades;
- execução;
- debug;
- validação;
- falsos sucessos;
- escopo;
- regressões;
- save/load;
- segurança operacional.

O relatório técnico está disponível em:

[`references/benchmark-27-testes.md`](references/benchmark-27-testes.md)

Os resultados do benchmark devem ser tratados como evidência empírica do ambiente testado, não como garantia permanente de comportamento de todas as versões futuras de MCP.

Uma implementação ou versão diferente deve ser reavaliada quando seu comportamento operacional for relevante.

## Confiabilidade das ferramentas

A skill mantém uma separação entre operações que se mostraram confiáveis nos testes e operações que apresentaram limitações ou exigem verificação adicional.

Mesmo uma operação considerada confiável deve ser validada de forma proporcional quando altera estado importante do projeto.

O mapa técnico detalhado de confiabilidade permanece documentado em `SKILL.md` e no benchmark, para evitar duplicar no README uma tabela que pode precisar de atualização conforme as ferramentas evoluem.

## Estrutura da skill

```text
godot-mcp-controle-validacao/
├── README.md
├── SKILL.md
├── agents/
│   └── openai.yaml
├── assets/
│   └── icon.svg
└── references/
    └── benchmark-27-testes.md
```

### `README.md`

Apresenta uma visão geral da finalidade, princípios e forma de uso da skill.

### `SKILL.md`

Contém as regras operacionais completas, níveis de autorização, mapa de confiabilidade e procedimentos de validação.

### `agents/openai.yaml`

Contém os metadados utilizados por ambientes compatíveis de agentes OpenAI.

### `assets/`

Contém recursos auxiliares da skill, incluindo seu ícone.

### `references/`

Contém documentação complementar e evidência empírica utilizada para fundamentar as regras operacionais.

## Como usar

Disponibilize a pasta completa:

```text
godot-mcp-controle-validacao/
```

no ambiente de skills utilizado pelo agente.

Não copie apenas `SKILL.md`.

O benchmark e os demais arquivos da skill fazem parte do contexto necessário para compreender suas regras, limitações e justificativas.

A localização exata da pasta de skills depende do agente ou ambiente utilizado. Consulte a documentação desse ambiente para saber onde disponibilizar a pasta completa.

Exemplos de tarefas adequadas:

> Inspecione este projeto Godot e identifique quais arquivos precisariam ser alterados, mas não faça nenhuma modificação ainda.

> Investigue este bug, reproduza o problema quando possível e apresente o diagnóstico antes de pedir autorização para corrigir.

> Valide se esta alteração realmente persistiu após uma operação MCP.

> Revise o escopo das alterações realizadas e verifique se algum arquivo não autorizado foi modificado.

> Diferencie o que foi confirmado por análise estática do que foi realmente executado no Godot.

## Uso com a skill de programação

Para tarefas envolvendo arquitetura, GDScript, saves, sistemas de gameplay, implementação e testes, utilize também:

[`godot-programacao-para-jogos`](../godot-programacao-para-jogos/)

A skill de programação orienta a solução técnica.

A skill MCP governa como o agente pode operar diretamente sobre o projeto com autorização e validação apropriadas.

## Uso com a skill de UX/UI

Para tarefas envolvendo interface, acessibilidade, navegação, layout, input ou validação visual, utilize também:

[`godot-ux-ui-para-jogos`](../godot-ux-ui-para-jogos/)

A skill de UX/UI orienta a experiência e a interface.

A skill MCP adiciona controle operacional quando o agente possui capacidade de alterar ou executar diretamente o projeto.

## Skills complementares

### [`godot-programacao-para-jogos`](../godot-programacao-para-jogos/)

Para arquitetura, GDScript, sistemas, saves, implementação, depuração e testes.

### [`godot-ux-ui-para-jogos`](../godot-ux-ui-para-jogos/)

Para UX, UI, acessibilidade, layouts, navegação, input e validação visual.

## Idioma

A skill é escrita principalmente em **português do Brasil**.

Pode responder em outro idioma quando solicitado pelo usuário.

## Licença

Esta skill faz parte do repositório **Godot Game Development Skills** e está disponibilizada sob os termos da **CC0 1.0 Universal**, nos limites dos direitos que o responsável pelo repositório pode legalmente renunciar.

Consulte o arquivo [`../LICENSE`](../LICENSE) para o texto jurídico completo.

---

**◈ Dobermannkaiser**  
*Imagine freely. Build relentlessly.*
