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
