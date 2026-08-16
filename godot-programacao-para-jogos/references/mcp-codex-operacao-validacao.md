# Codex + MCP Godot: operação e validação

## Sumário

1. Papel de cada camada
2. Configuração e descoberta
3. Capacidades empiricamente confiáveis
4. Limitações e falsos sucessos
5. Autorização e escopo
6. Fluxo para bugs
7. Fluxo para implementação
8. Validação independente
9. Testes e ferramentas temporárias
10. Checklist

## 1. Papel de cada camada

Usar uma divisão explícita:

```text
Codex/acesso direto
├── ler e editar .gd, .tscn, .tres e project.godot
├── pesquisar call sites e referências
├── calcular hashes e inspecionar diffs
└── criar testes e fixtures autorizados

MCP Godot
├── reconhecer projeto e versão
├── abrir editor quando necessário
├── executar projeto/cena
├── coletar output do motor real
└── parar a execução

Godot real
└── prova parser, importação, lifecycle e runtime observados
```

Não usar MCP como editor principal de GDScript. Para conteúdo textual e Variants complexos, preferir edição direta verificável. MCP é ponte para o motor, não autoridade sobre o resultado.

## 2. Configuração e descoberta

Uma configuração observada com `@coding-solo/godot-mcp` usa `npx -y @coding-solo/godot-mcp` e `GODOT_PATH` apontando para o executável do Godot, nunca para a pasta do projeto. O projeto é determinado pelo workspace/pasta selecionada.

Não presumir nomes de ferramentas. Descobrir as expostas na sessão. Capacidades observadas incluem:

- `get_project_info`, `list_projects`, `get_godot_version`;
- `launch_editor`, `run_project`, `stop_project`, `get_debug_output`;
- `create_scene`, `add_node`, `load_sprite`, `save_scene`;
- `export_mesh_library`, `get_uid`, `update_project_uids`.

Não instalar, baixar ou atualizar servidor MCP, Node, Godot, templates, addons ou dependências sem autorização. `GODOT_PATH` deve corresponder à versão-alvo do projeto.

## 3. Capacidades empiricamente confiáveis

Em benchmark progressivo com Godot 4.7.1 e 27 testes, funcionaram de modo consistente:

- conexão, reconhecimento do projeto e versão;
- abrir editor, executar projeto, obter debug e parar execução;
- criar hierarquia de nós e persistir propriedades simples;
- `String`, `bool`, `int`, `float` e enums por inteiro;
- `Texture2D` por `load_sprite` quando o nó alvo existe;
- lotes de 11 propriedades simples;
- cenas de UI com dezenas de propriedades simples;
- stress de 50 e 200 `Label` sem perda de conexão, embora com degradação de velocidade;
- captura de parser error, correção, reexecução e validação;
- reprodução de bugs lógicos e bugs atravessando múltiplos scripts;
- execução de regressões e round trips de estado real.

Esses resultados são evidência para a combinação testada, não garantia universal para toda versão do servidor, plataforma, Godot ou tipo Variant. Não repetir stress já demonstrado sem hipótese nova.

## 4. Limitações e falsos sucessos

Falhas confirmadas:

- `create_scene` pode criar a raiz com nome `root` em vez do nome solicitado;
- `Vector2` enviado por propriedades genéricas pode não persistir;
- `Color` pode persistir como preto mesmo com retorno aparente de sucesso;
- parent inexistente pode retornar sucesso sem criar corretamente;
- `load_sprite` em nó inexistente pode retornar sucesso;
- propriedade inexistente pode ser ignorada silenciosamente;
- outros Variants complexos permanecem não confiáveis sem reteste específico.

Consequências:

1. nunca usar a resposta `success` como pós-condição;
2. após write MCP, reler `.tscn`, `.tres`, `.gd` ou configuração persistida;
3. validar nome, tipo, parent, owner, propriedade e valor exato;
4. para `Vector2`, `Color`, Resources, Dictionaries e outros Variants complexos, preferir edição direta ou API especializada testada;
5. executar o motor depois da verificação de arquivo;
6. rejeitar a hipótese se a reprodução não confirmar o problema.

## 5. Autorização e escopo

Antes de qualquer write em produção:

1. investigar;
2. reproduzir quando tecnicamente possível;
3. registrar expected e actual;
4. apresentar evidência;
5. diagnosticar a causa provável;
6. listar arquivos e funções envolvidos;
7. propor a solução mínima e testes;
8. aguardar autorização explícita.

Autorização é literal e por escopo. Uma pasta de testes não libera código de produção. Um script autorizado não libera cena, asset, `project.godot`, preset, save ou outro manager. Se surgir nova necessidade, parar, explicar e pedir nova autorização.

Não modificar saves reais de forma destrutiva. Não promover versão estável sem validação humana quando necessária e aprovação explícita do usuário.

## 6. Fluxo para bugs

Usar:

```text
relato
→ comportamento esperado observável
→ inspeção da arquitetura
→ reprodução no código real
→ expected/actual e evidência
→ hipótese estreita
→ causa
→ solução mínima
→ autorização
→ write
→ releitura/diff
→ mesma reprodução
→ casos adjacentes
→ regressões afetadas
→ Godot real
→ validação humana aplicável
```

Se a primeira hipótese não reproduzir, rejeitá-la. Não alterar produção para “achar algum bug”. Para bugs de save, usar round trip, JSON quando aplicável, múltiplos round trips e determinismo. Quando o sintoma surge após repetições, preservar snapshots e diff semântico em `R0` (origem), `R1`, `R2`, `R3` e `R4` de controle, com a mesma fixture e seed; localizar a primeira transição divergente em vez de comparar apenas início e fim. Para bugs de UI, testar input e foco reais, não apenas visibilidade.

## 7. Fluxo para implementação

Estruturar tarefas com quatro blocos enxutos:

- **Goal:** resultado funcional e motivo;
- **Context:** versão, sintomas, arquivos/sistemas prováveis e evidência;
- **Constraints:** escopo autorizado, compatibilidade e proibições;
- **Done when:** reprodução, regressões, output real e critérios observáveis.

Não repetir guardrails inteiros em todo prompt quando esta skill já os fornece. Não superespecificar singleton, flag, `mouse_filter`, `call_deferred` ou estrutura antes de ler a arquitetura.

## 8. Validação independente

Antes de abrir editor ou executar numa fase declarada somente leitura, identificar destinos que o motor pode gravar: `.godot/`, import cache, logs, `user://`, configurações, autosave, telemetria e arquivos produzidos por ferramentas internas. Preferir cópia temporária descartável da base, diretório de usuário isolado e autosave/telemetria suspensos. Registrar baseline de árvore/hash/mtime dos alvos relevantes e comparar depois. Se isolamento suficiente não estiver disponível, limitar a etapa à inspeção estática ou pedir autorização para os efeitos inevitáveis; não chamar runtime potencialmente gravável de “zero writes”.

Depois de cada write:

1. reler conteúdo persistido;
2. inspecionar diff e lista de arquivos modificados;
3. confirmar ausência de mudanças fora do escopo;
4. executar análise estática proporcional;
5. rodar a reprodução original;
6. executar regressões diretamente afetadas;
7. usar integração quando houver estado ou sistemas cruzados;
8. iniciar a cena principal e coletar parser/runtime errors;
9. parar processos do Godot;
10. declarar o que exige validação visual, auditiva, física ou humana.

Auditar também efeitos colaterais não textuais: save/configuração, project settings, recursos importados, cache e arquivos externos. Releitura de `.gd`/`.tscn` não basta quando a execução pode escrever fora deles.

MCP confirma que uma chamada foi aceita; arquivo persistido confirma o write; Godot confirma o comportamento executado; o usuário confirma experiência subjetiva e gameplay completo quando necessário.

## 9. Testes e ferramentas temporárias

Criar teste persistente somente com autorização. Ferramenta temporária deve:

- operar em memória ou cópia isolada;
- não tocar save real;
- suspender autosave quando aplicável;
- restaurar estado em saída garantida;
- usar produção real em vez de duplicar a regra;
- registrar expected/actual e marcador inequívoco;
- ser removida se não fizer parte da suíte aprovada.

Não preservar features experimentais só porque passaram. Separar correção real, experimento e teste descartável antes de consolidar uma versão.

## 10. Checklist

- [ ] Projeto, versão do motor e cena principal confirmados.
- [ ] Base estável e escopo autorizado registrados.
- [ ] MCP usado como ponte para execução, não como prova de persistência.
- [ ] Variants complexos não enviados por caminho genérico sem reteste.
- [ ] Bug reproduzido ou hipótese explicitamente não confirmada.
- [ ] Runtime somente leitura foi isolado e seus destinos graváveis auditados, ou permaneceu pendente.
- [ ] Arquivos exatos autorizados antes do write.
- [ ] Arquivos persistidos relidos e diff inspecionado.
- [ ] Mesma reprodução e regressões afetadas executadas.
- [ ] Output real do Godot coletado e classificado.
- [ ] Processos do Godot encerrados.
- [ ] Saves reais preservados.
- [ ] Validação humana e promoção de versão não foram inventadas.
