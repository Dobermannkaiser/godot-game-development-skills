# Godot — UX e UI para Jogos

**Skill em português para UX, UI e acessibilidade em jogos desenvolvidos com Godot 4.x.**

**Versão atual:** `4.0.0`

Esta skill foi criada para orientar agentes de inteligência artificial no planejamento, implementação, revisão e validação de interfaces e experiências de usuário em projetos Godot.

Ela trata UX e UI como partes funcionais do jogo, conectando clareza, navegação, acessibilidade, input, responsividade e apresentação aos sistemas reais do projeto.

## Quando usar

Use esta skill em tarefas relacionadas a:

- UX de jogos;
- UI e HUD;
- menus e configurações;
- navegação;
- modais;
- foco;
- mouse;
- teclado;
- controle;
- toque;
- acessibilidade;
- responsividade;
- localização;
- layouts;
- temas;
- componentes reutilizáveis;
- feedback visual;
- validação da interface;
- integração de UI com sistemas Godot.

## Princípios principais

### Começar pela tarefa do jogador

A interface deve ser projetada a partir do que o jogador precisa compreender, decidir e fazer.

A aparência visual não deve esconder estado, consequência, custo, foco, navegação ou saída.

### Inspecionar antes de redesenhar

Antes de propor mudanças, o agente deve compreender a interface existente, incluindo:

- cenas;
- scripts;
- temas;
- resolução;
- plataformas;
- métodos de entrada;
- estados;
- saves;
- localização;
- tutorial;
- comportamento já aprovado.

### Preservar a identidade visual

A skill não recomenda aplicar tendências visuais apenas por serem populares.

Minimalismo, animações, glassmorphism ou outros estilos só devem ser utilizados quando realmente ajudarem a experiência.

### Tratar acessibilidade como requisito funcional

Acessibilidade deve fazer parte do projeto desde o início.

Isso inclui questões como:

- contraste;
- escala;
- foco;
- redução de movimento;
- alternativas de entrada;
- clareza de linguagem;
- localização;
- suporte apropriado a diferentes formas de interação.

### Separar UI das regras do jogo

A interface apresenta informações e solicita ações.

As regras do domínio continuam responsáveis por validar decisões e alterar o estado do jogo.

A UI então apresenta o resultado atualizado.

### Validar de forma proporcional

Uma interface não deve ser considerada funcional apenas porque seus arquivos parecem corretos.

Sempre que o ambiente permitir, a validação deve considerar navegação, foco, estados, resolução, inputs, acessibilidade e comportamento em runtime.

## Fluxo geral

O fluxo recomendado pela skill pode ser representado assim:

```text
entender o objetivo do jogador
        ↓
inspecionar a interface existente
        ↓
mapear informação, decisão e ação
        ↓
definir estados e navegação
        ↓
projetar estrutura e hierarquia
        ↓
implementar componentes
        ↓
validar inputs e responsividade
        ↓
validar acessibilidade e estados
        ↓
revisar experiência e limitações
```

O objetivo é evitar que o polimento visual seja realizado antes de a estrutura da experiência estar correta.

## Áreas cobertas

### Pesquisa e fluxo

A skill inclui orientação para:

- jornadas;
- arquitetura de informação;
- hierarquia;
- onboarding;
- progressive disclosure;
- UX writing.

### Design system e layout

Abrange:

- `Theme`;
- tokens;
- componentes;
- tipografia;
- containers;
- resoluções;
- safe areas;
- escala;
- layouts responsivos.

### Godot UI

Inclui práticas relacionadas a:

- `Control`;
- `Container`;
- input;
- foco;
- modais;
- scroll;
- tooltips;
- navegação;
- GDScript de apresentação.

### Acessibilidade e localização

Inclui atenção a:

- contraste;
- movimento;
- acessibilidade visual;
- acessibilidade motora;
- acessibilidade cognitiva;
- legendas;
- localização;
- pseudolocalização;
- RTL quando aplicável.

### Testes e validação

A skill orienta validações relacionadas a:

- regressão visual;
- estados extremos;
- conteúdo longo;
- diferentes resoluções;
- foco;
- mouse;
- teclado;
- controle;
- toque;
- runtime;
- inspeção visual;
- testes humanos quando disponíveis.

## Estrutura da skill

```text
godot-ux-ui-para-jogos/
├── README.md
├── SKILL.md
├── agents/
│   └── openai.yaml
├── assets/
│   └── icon.svg
└── references/
```

### `README.md`

Apresenta uma visão geral da skill para leitura humana, incluindo seu propósito, áreas cobertas e relação com as outras skills.

### `SKILL.md`

Contém as regras, princípios e fluxo operacional principal da skill.

### `agents/openai.yaml`

Contém os metadados de integração utilizados por ambientes compatíveis de agentes OpenAI.

### `assets/`

Contém recursos auxiliares, incluindo o ícone da skill.

### `references/`

Contém documentação detalhada sobre UX, UI, acessibilidade, Godot e validação.

Os principais temas incluem:

- fundamentos de UX e conteúdo;
- design system e responsividade;
- controles, input e foco;
- acessibilidade e localização;
- padrões de telas e feedback;
- testes e diagnóstico;
- uso de Codex e Godot MCP para validação de UI.

## Como usar

Disponibilize a pasta completa:

```text
godot-ux-ui-para-jogos/
```

no diretório de skills utilizado pelo seu agente ou ambiente de desenvolvimento.

Não copie somente `SKILL.md`.

Os arquivos em `references/`, além dos metadados e recursos complementares, fazem parte do conjunto de conhecimento utilizado pela skill.

A localização exata da pasta de skills depende do agente ou ambiente utilizado. Consulte a documentação desse ambiente para saber onde disponibilizar a pasta completa.

Exemplos de tarefas adequadas:

> Analise a navegação e a hierarquia desta interface antes de sugerir mudanças.

> Revise este menu para mouse, teclado e controle.

> Verifique problemas de foco e comportamento modal nesta tela.

> Avalie esta interface em relação a acessibilidade e responsividade.

> Implemente esta UI preservando o comportamento e a identidade visual existentes.

## Validação de interface

Uma interface pode parecer correta em uma análise estática e ainda apresentar problemas durante o uso real.

Quando aplicável, a validação deve considerar diferentes camadas:

```text
estrutura
   ↓
conteúdo
   ↓
navegação e foco
   ↓
inputs
   ↓
responsividade
   ↓
acessibilidade
   ↓
runtime
   ↓
inspeção visual
```

Essas formas de validação não devem ser tratadas como equivalentes.

Por exemplo, verificar uma cena ou script não substitui testar foco, navegação, resolução ou comportamento visual durante a execução.

Quando alguma validação não puder ser realizada, essa limitação deve ser declarada.

## Uso com Codex e Godot MCP

Quando o agente possui acesso direto ao projeto, a skill recomenda separar claramente:

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
validação da interface
```

A capacidade de editar cenas, scripts ou temas diretamente não significa autorização automática para realizar essas mudanças.

Para regras específicas de autorização, controle operacional e validação de operações MCP, utilize também:

[`godot-mcp-controle-validacao`](../godot-mcp-controle-validacao/)

## Integração com programação

Mudanças de UI frequentemente dependem de sistemas técnicos existentes.

Quando uma alteração exigir mudanças substanciais em:

- arquitetura;
- GDScript;
- saves;
- sistemas de gameplay;
- regras de domínio;
- integração entre sistemas;

utilize também:

[`godot-programacao-para-jogos`](../godot-programacao-para-jogos/)

A skill de UX/UI orienta a experiência e a interface, enquanto a skill de programação complementa a implementação e integração técnica.

## Skills complementares

### [`godot-programacao-para-jogos`](../godot-programacao-para-jogos/)

Para arquitetura, GDScript, sistemas de jogo, saves, integração técnica ampla e testes.

### [`godot-mcp-controle-validacao`](../godot-mcp-controle-validacao/)

Para autorização, execução controlada via MCP, validação independente e segurança operacional.

## Idioma

A skill é escrita principalmente em **português do Brasil**.

Pode responder em outro idioma quando solicitado pelo usuário.

## Licença

Esta skill faz parte do repositório **Godot Game Development Skills** e está disponibilizada sob os termos da **CC0 1.0 Universal**, nos limites dos direitos que o responsável pelo repositório pode legalmente renunciar.

Consulte o arquivo [`../LICENSE`](../LICENSE) para o texto jurídico completo.

---

**◈ Dobermannkaiser**  
*Imagine freely. Build relentlessly.*
