# Godot Game Development Skills

Três skills em português para auxiliar agentes de inteligência artificial no desenvolvimento de jogos com Godot 4.x, cobrindo programação, UX/UI e operação segura com Codex + Godot MCP.

## Skills incluídas

### Godot — Programação para Jogos 4.0

Programação, arquitetura, depuração, testes, saves, migrações, sistemas narrativos, relacionamentos, economia, integração com Codex e Godot MCP e entrega de projetos em Godot 4.x.

A skill também define práticas de autorização por escopo, validação independente de alterações, preservação de bases estáveis e uso do Godot real para confirmar parser, runtime e comportamento.

### Godot — UX e UI para Jogos 4.0

Planejamento, implementação e revisão de UX/UI, acessibilidade, responsividade, navegação, conteúdo procedural e validação visual em Godot 4.x.

Inclui orientação para integração técnica com sistemas do jogo, foco, input, responsividade, clareza de interface e validação da experiência do usuário.

### Godot MCP — Controle, Segurança e Validação 1.0

Camada operacional para uso seguro de Codex, acesso direto aos arquivos e Godot MCP.

Define regras para:

* autorização explícita antes de alterações;
* inspeção antes de edição;
* validação independente de writes;
* prevenção de falsos sucessos do MCP;
* reprodução de bugs antes da correção;
* regressões;
* testes de save/load e round-trip;
* auditoria de escopo;
* uso responsável de ferramentas destrutivas ou de versionamento.

As regras dessa skill foram derivadas de um benchmark progressivo de 27 testes realizados com Godot MCP no projeto Golem’s Mandate.

## Estrutura

Cada skill é distribuída como uma pasta completa.

As skills de programação e UX/UI podem incluir:

* `SKILL.md`;
* metadados em `agents/`;
* ícone em `assets/`;
* documentação modular em `references/`.

A skill de controle MCP inclui:

* `SKILL.md`;
* `README.md`;
* documentação técnica em `references/`.

A estrutura pode evoluir entre versões conforme a necessidade de cada skill.

## Instalação

Copie a pasta completa da skill para o diretório de skills compatível com seu agente ou ferramenta.

Não copie somente o arquivo `SKILL.md`, pois os arquivos complementares fazem parte das instruções e referências utilizadas pela skill.

## Skills disponíveis

* `godot-programacao-para-jogos` — versão 4.0.0
* `godot-ux-ui-para-jogos` — versão 4.0.0
* `godot-mcp-controle-validacao` — versão 1.0.0

## Uso conjunto

As três skills podem ser utilizadas em conjunto.

Uma divisão recomendada é:

* `godot-programacao-para-jogos` — arquitetura, GDScript, sistemas, saves, implementação e testes;
* `godot-ux-ui-para-jogos` — UX/UI, acessibilidade, layout, navegação e validação visual;
* `godot-mcp-controle-validacao` — autorização, controle operacional, execução via MCP, validação independente e segurança.

A skill MCP não substitui as skills de programação ou UX/UI. Ela funciona como uma camada adicional de controle e validação quando uma IA possui acesso direto ao projeto ou utiliza Godot MCP.

## Inteligência artificial

Estas skills foram produzidas com auxílio extensivo de ferramentas de inteligência artificial.

O responsável pelo repositório não reivindica que todo o material tenha sido criado manualmente por ele.

## Licença

Na medida permitida por lei, todos os direitos autorais e direitos conexos que o responsável por este repositório possa possuir sobre este material são dedicados ao domínio público por meio da **CC0 1.0 Universal**.

Você pode copiar, modificar, distribuir e utilizar estas skills, inclusive para fins comerciais, sem pedir autorização e sem obrigação de atribuição.

Consulte o arquivo [`LICENSE`](LICENSE) para ler o texto jurídico completo da CC0 1.0 Universal.

A CC0 aplica-se somente aos direitos que o responsável por este repositório efetivamente possui e pode legalmente renunciar. Materiais de terceiros, caso existam, não são abrangidos por essa dedicação e permanecem sujeitos aos direitos e às licenças de seus respectivos titulares.

Nenhuma garantia é fornecida sobre estas skills ou sua adequação a uma finalidade específica.
