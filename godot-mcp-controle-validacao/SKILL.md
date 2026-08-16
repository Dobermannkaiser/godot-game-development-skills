---
name: godot-mcp-controle-validacao
display_name: Godot MCP — Controle, Segurança e Validação
version: 1.0.0
language: pt-BR
description: Controlar o uso de Godot MCP e Codex em projetos Godot 4.x com autorização obrigatória antes de qualquer alteração, inspeção prévia, validação independente de writes, prevenção de falsos sucessos, reprodução antes de corrigir bugs, regressões, auditoria de escopo e diretrizes empiricamente derivadas de um benchmark de 27 testes no Golem’s Mandate. Usar sempre que uma IA operar Godot via MCP, editar projeto com Codex ou combinar acesso direto aos arquivos com execução/debug no motor.
---

# Godot MCP — Controle, Segurança e Validação

## Missão

Esta skill governa **como** uma IA deve usar **Codex + acesso direto aos arquivos + Godot MCP** de forma segura, verificável e reversível.

Ela não substitui uma skill de programação Godot ou de UX/UI. Ela funciona como camada de **controle operacional, autorização, validação e segurança**.

Esta é a versão **1.0.0** da skill. Responder em português do Brasil, salvo pedido em outro idioma.

Quando disponível, usar em conjunto com:

- `godot-programacao-para-jogos` — arquitetura, GDScript, saves, sistemas e implementação;
- `godot-ux-ui-para-jogos` — UX/UI, acessibilidade, layout, interação e validação visual.

Para detalhes empíricos completos dos 27 testes, ler:

```text
references/benchmark-27-testes.md
```

---

# 1. Princípios inegociáveis

## 1.1. Autorização obrigatória antes de qualquer alteração

**Nunca escrever, criar, editar, apagar, mover, renomear ou substituir arquivos do projeto sem pedir autorização explícita imediatamente antes da primeira alteração.**

Isso vale mesmo quando:

- a correção parece óbvia;
- o usuário já pediu “corrija” ou “implemente”;
- a alteração é pequena;
- o arquivo está em `tests/`;
- o MCP afirma que a operação é segura;
- a IA já alterou arquivos semelhantes antes.

O fluxo obrigatório é:

```text
inspecionar
→ explicar o que pretende alterar
→ listar o escopo
→ pedir autorização
→ aguardar resposta positiva
→ só então escrever
```

Uma autorização pode cobrir várias alterações **somente quando o escopo estiver claramente listado**.

### Modelo de solicitação

```text
AUTORIZAÇÃO NECESSÁRIA

Pretendo alterar:
- <arquivo/pasta 1>
- <arquivo/pasta 2>

Objetivo:
<resumo>

Tipo de alteração:
<criar/editar/remover/mover>

Validação prevista:
<testes/MCP/regressão>

Nenhum outro arquivo de produção será modificado sem nova autorização.

Você autoriza essas alterações?
```

## 1.2. Autorização não é transitiva

Autorização para:

```text
Arquivo A
```

não autoriza:

```text
Arquivo B
```

se B não estava no escopo.

Se um segundo problema aparecer durante a tarefa e exigir novo production code:

1. parar;
2. preservar evidência;
3. explicar o novo problema;
4. pedir nova autorização.

## 1.3. Inspeção antes de edição

Antes de solicitar autorização de escrita:

- identificar projeto e versão;
- ler `project.godot` quando relevante;
- localizar scripts/cenas afetados;
- procurar call sites;
- localizar testes existentes;
- identificar save/schema se a mudança tocar estado;
- distinguir production code de harness/teste.

Nunca inventar API que não foi encontrada no código real.

## 1.4. Não confundir “funcionou” com “está aprovado para o produto”

Uma feature experimental pode passar em todos os testes e ainda assim não pertencer ao jogo.

Separar sempre:

```text
funciona tecnicamente
≠
foi aprovado como requisito de design
```

## 1.5. Sucesso do MCP não é prova

A mensagem `success` do MCP é apenas uma indicação de que a chamada terminou.

Ela **não prova**:

- que a propriedade persistiu;
- que o node foi adicionado;
- que o valor ficou correto;
- que a textura foi aplicada;
- que o jogo se comporta como esperado.

Depois de writes relevantes, exigir evidência independente.

## 1.6. Não afirmar execução que não ocorreu

Distinguir claramente:

- análise estática;
- leitura de arquivo;
- execução MCP;
- output real do Godot;
- teste humano.

Nunca declarar “validado no Godot” se não houve execução real.

Nunca declarar “validado pelo usuário” se o usuário não confirmou.

## 1.7. Não promover base estável por conta própria

Uma IA pode dizer:

```text
candidata tecnicamente validada
```

mas a promoção para base estável oficial depende de aprovação explícita do usuário.

---

# 2. Níveis operacionais e autorização

## Nível 0 — somente leitura

Exemplos:

- ler scripts/cenas;
- pesquisar referências;
- listar call sites;
- calcular hash;
- comparar conteúdo;
- analisar logs já existentes;
- `get_project_info`;
- `get_godot_version`.

Não altera o projeto.

Pode ser feito sem autorização de escrita quando fizer parte da análise pedida.

## Nível 1 — execução do Godot

Exemplos:

- `launch_editor`;
- `run_project`;
- `get_debug_output`;
- `stop_project`.

A execução pode tocar caches `.godot` e, dependendo da cena, `user://`.

Regra:

- se o usuário pediu explicitamente para executar/testar, isso autoriza a execução dentro daquele escopo;
- se a tarefa era apenas análise, pedir autorização antes de iniciar execução pesada ou potencialmente mutável;
- nunca interagir destrutivamente com saves reais sem autorização específica.

## Nível 2 — escrita de projeto

Exemplos:

- editar `.gd`;
- editar `.tscn`;
- `create_scene`;
- `add_node`;
- `load_sprite` que salva cena;
- criar probe/teste;
- mudar `project.godot`.

**Sempre exige autorização explícita prévia.**

## Nível 3 — operação destrutiva ou de versionamento

Exemplos:

- apagar/mover/renomear arquivos;
- limpeza massiva;
- reset/restore/checkout/clean;
- migração de save;
- mudança de schema;
- remoção de assets;
- reestruturação de pastas;
- promoção de versão.

Exige autorização explícita e escopo detalhado. Se a operação não estava claramente autorizada, parar.

---

# 3. Mapa de confiabilidade do Godot MCP

Este mapa é derivado do benchmark de 27 testes. Tratar como referência operacional até que uma nova versão do MCP seja retestada.

## 3.1. Confiável nos testes

- `get_project_info`;
- `get_godot_version`;
- `launch_editor`;
- `run_project`;
- `get_debug_output`;
- `stop_project`;
- `create_scene` para cena + tipo da raiz;
- `add_node` com parent válido;
- `save_scene`;
- propriedades `String`;
- propriedades `bool`;
- propriedades `int`;
- propriedades `float`;
- enums representados por inteiro;
- batch de propriedades primitivas;
- `load_sprite` quando o `Sprite2D` já existe;
- estruturas com 50 e 200 nodes, embora lentas.

## 3.2. Falhas/limitações conhecidas

### `Vector2` em properties

Quatro representações diferentes foram aceitas pelo MCP e reportadas como sucesso, mas **nenhuma persistiu**.

**Regra:** não usar generic `properties` para `Vector2` sem reteste específico. Editar `.tscn`/GDScript diretamente.

### `Color` em properties

Quatro representações diferentes foram aceitas, mas persistiram como:

```text
Color(0, 0, 0, 1)
```

independentemente da cor pedida.

**Regra:** não usar generic `properties` para `Color`. Editar arquivo diretamente.

### Nome da raiz em `create_scene`

O tipo da raiz é configurável; o nome não foi exposto e ficou `root`.

**Regra:** se o nome importar, editar o `.tscn` diretamente depois de autorização.

### Parent inexistente

Pode retornar falso sucesso e não persistir node.

### `load_sprite` em node inexistente

Pode informar sucesso sem aplicar a textura.

### Propriedade inexistente

Pode salvar o node e propriedades válidas, ignorar a inválida e ainda reportar sucesso geral.

---

# 4. Regra de pós-verificação obrigatória

Após qualquer write MCP, executar pelo menos uma destas verificações, e em mudanças relevantes preferir várias:

1. **Arquivo persistido:** ler o `.tscn`/`.gd` e confirmar estrutura/propriedades.
2. **Execução real:** rodar a cena/probe no Godot.
3. **Marker explícito:** exigir `_OK`/`_FAILED` com expected/actual.
4. **Regressão:** executar teste relacionado.
5. **Auditoria:** confirmar arquivos modificados/diff/hash.

Se MCP diz “success” mas a evidência independente não aparece, classificar como:

```text
NÃO VERIFICADO
```

ou:

```text
FALHA
```

Nunca como sucesso.

---

# 5. Escolha correta entre MCP e edição direta

## Usar MCP preferencialmente para

- abrir o Godot;
- executar cena/projeto;
- capturar debug;
- parar execução;
- criar estruturas pequenas com propriedades simples;
- carregar textura em node existente;
- verificar comportamento no motor.

## Usar acesso direto aos arquivos preferencialmente para

- GDScript;
- `.tscn` com tipos complexos;
- `Vector2`;
- `Color`;
- transforms;
- Resources complexos;
- alterações grandes/repetitivas;
- refactors;
- diff preciso;
- inspeção de call sites;
- hashes e auditoria.

Depois da edição direta, usar MCP para validar no Godot.

---

# 6. Performance e stress

Benchmark observado:

```text
50 Labels  → 50/50 corretos, ~5,27 s/node
200 Labels → 200/200 corretos, ~6,23 s/node
```

A sequência de 200 levou aproximadamente 21 minutos e apresentou degradação moderada de desempenho.

**Regra:** não usar MCP node-a-node em massa apenas porque é possível. Quando a estrutura é grande e determinística, preferir geração/edição direta + validação.

---

# 7. RID/resource leak warnings

Cenas com texto/visual produziram warnings de teardown envolvendo, entre outros:

- `ShapedTextDataAdvanced`;
- `FontAdvanced`;
- algumas referências de textura/RendererDummy.

Nos benchmarks não houve corrupção ou perda de nodes associada a esses warnings.

**Regra:**

- registrar o warning;
- não confundir com falha funcional automaticamente;
- investigar se aparecer durante runtime normal do jogo e não apenas no encerramento dos probes/MCP.

---

# 8. Protocolo obrigatório para corrigir bugs desconhecidos

Não alterar production code antes de uma reprodução real quando o bug ainda for hipótese.

Fluxo:

```text
1. investigar
2. formular hipótese
3. criar reprodução em tests/ após autorização
4. executar Godot
5. observar expected versus actual
6. confirmar BUG_REPRODUZIDO
7. diagnosticar causa raiz
8. pedir autorização para o patch de production
9. corrigir minimamente
10. executar exatamente o mesmo cenário
11. adicionar casos diferentes
12. executar regressão
13. auditar escopo
```

Se o candidato não reproduzir:

```text
CANDIDATO_REJEITADO
```

Não modificar production code para fazer a hipótese se tornar verdadeira.

---

# 9. Anti-trapaça dos testes

É proibido:

- alterar expected depois de observar actual;
- hardcodar resultado do probe;
- adicionar condição específica para nome do teste;
- remover caso que falha;
- imprimir `_OK` sem validar estado;
- modificar o teste para aceitar o comportamento quebrado;
- copiar uma implementação de production para dentro do probe e testar a cópia.

O probe deve exercer **production code real**.

---

# 10. Protocolo para bug lógico

Ausência de Parser Error ou runtime error **não** significa sucesso.

Todo teste lógico deve ter:

```text
expected=<valor>
actual=<valor>
```

ou invariantes equivalentes.

Se o resultado estiver errado, o agente deve rastrear a causa no código, mesmo quando `finalErrors=[]`.

---

# 11. Protocolo multiarquivo

Quando o sintoma aparece em arquivo A e a causa pode estar em B:

1. identificar a chamada real;
2. seguir referências/preloads;
3. localizar função responsável;
4. não corrigir o probe se o bug estiver no dependency;
5. restringir o patch ao menor conjunto de production files possível;
6. reexecutar o mesmo probe.

---

# 12. Protocolo de regressão

Antes de corrigir um comportamento compartilhado, identificar quais casos já funcionam.

Depois da correção, reportar:

```text
testes corretos antes: X/N
testes corretos depois: Y/N
regressões introduzidas: Z
```

Uma correção que resolve o novo caso mas quebra comportamento previamente correto não está validada.

---

# 13. Protocolo de save/load e round-trip

Qualquer alteração em estado persistente deve avaliar, quando aplicável:

```text
runtime setter/API
→ export
→ nova instância
→ import
→ comparação semântica
```

Perguntas obrigatórias:

1. runtime pode criar o estado?
2. export preserva?
3. import aceita o próprio estado exportado?
4. valores persistentes são semanticamente iguais?
5. campos derivados/transitórios foram excluídos corretamente?
6. `export → import → export` é estável?
7. múltiplos ciclos causam deriva?
8. duas reconstruções independentes são determinísticas?

Para mudanças relevantes, considerar:

- JSON round-trip;
- double round-trip;
- triple round-trip;
- determinismo.

---

# 14. Protocolo para integração entre sistemas

Antes de criar acoplamento direto:

1. ler os dois managers;
2. identificar responsabilidades;
3. verificar APIs públicas existentes;
4. procurar composição/bridge;
5. evitar dependência circular;
6. separar feature técnica de requirement de produto.

Uma bridge experimental deve ser removida depois do experimento se não houver requirement aprovado.

---

# 15. Suite permanente do Golem’s Mandate v3.11.6

Estado de referência ao fim do benchmark:

```text
tests/
├── regression/
│   ├── population_housing_capacity_roundtrip/
│   ├── story_custom_dialogue_roundtrip/
│   ├── building_queue_roundtrip/
│   ├── dialogue_manager_basic/
│   └── npc_relationship_manager_official_flow/
└── integration/
    └── game_state_roundtrip/
```

Markers esperados:

```text
POPULATION_ROUNDTRIP_OK
STORY_ROUNDTRIP_OK
BUILDING_QUEUE_ROUNDTRIP_OK
DIALOGUE_MANAGER_BASIC_OK
NPC_RELATIONSHIP_OFFICIAL_FLOW_OK
INTEGRATED_ROUNDTRIP_ALL_OK
INTEGRATED_TEST_ALL_OK
INTEGRATED_TRIPLE_ROUNDTRIP_OK
INTEGRATED_DETERMINISM_OK
```

Depois de uma mudança relacionada, executar os testes relevantes.

Para uma mudança ampla ou de save/state, executar também a integração completa.

---

# 16. Três fixes que pertencem à v3.11.6

Não remover sem nova evidência, novo teste e autorização explícita.

## PopulationState

Regra preservada:

```text
capacidade habitacional válida precisa respeitar múltiplos de 5
```

Proteção observada:

```gdscript
new_capacity % 5 != 0
```

## StoryManager

Regra preservada:

- contexto customizado não vazio aceito pela API precisa sobreviver ao round-trip;
- pending ID/context devem estar ambos vazios ou ambos preenchidos;
- não reintroduzir whitelist incompatível sem requirement explícito.

## BuildingManager

Regra preservada:

```text
housing target_level representa quantidade planejada de casas
```

A exigência de variante final não deve atingir housing:

```gdscript
not is_housing
```

---

# 17. Autorização para testes e harness

Criar um probe também é uma alteração.

Antes de criar arquivos em `tests/`, pedir autorização e explicar:

- pasta;
- finalidade;
- se é temporário ou permanente;
- production code exercitado;
- como será removido/consolidado depois.

Não deixar testes artificiais acumularem silenciosamente no projeto.

---

# 18. Proteção de save real

Por padrão:

- não sobrescrever save real do usuário;
- preferir dados em memória/probes;
- preferir serialização para memória/JSON quando suficiente;
- não escrever em `user://` real sem autorização específica.

Se o teste exige save físico, explicar risco e pedir autorização separada.

---

# 19. Baseline, hash e diff

Para mudanças de risco moderado/alto:

Antes:

- registrar hash dos production files afetados;
- registrar baseline dos testes.

Depois:

- mostrar diff resumido;
- listar funções modificadas;
- registrar novos hashes quando útil;
- listar todos os arquivos criados/modificados/removidos.

Se não existe Git, dizer claramente. Não inventar baseline.

---

# 20. Regras de escopo

Nunca fazer durante uma correção local sem autorização específica:

- refactor global;
- reformatar arquivos não relacionados;
- mudar versão do projeto;
- mudar save schema/version;
- atualizar Godot/addons;
- apagar caches/arquivos;
- limpar worktree;
- reorganizar pastas;
- remover assets.

Mudança incidental encontrada deve ser reportada, não absorvida silenciosamente.

---

# 21. Fluxo padrão de trabalho

```text
A. INSPECIONAR
   ↓
B. DEFINIR ESCOPO
   ↓
C. PEDIR AUTORIZAÇÃO
   ↓
D. BASELINE/HASH SE NECESSÁRIO
   ↓
E. ALTERAR MINIMAMENTE
   ↓
F. VERIFICAR ARQUIVO REAL
   ↓
G. EXECUTAR GODOT VIA MCP
   ↓
H. LER DEBUG/EXPECTED/ACTUAL
   ↓
I. REGRESSÃO
   ↓
J. INTEGRAÇÃO SE APLICÁVEL
   ↓
K. MAIN SCENE SE APLICÁVEL
   ↓
L. AUDITAR DIFF/ESCOPO
   ↓
M. RELATAR
   ↓
N. USUÁRIO DECIDE PROMOÇÃO
```

---

# 22. Critérios de conclusão

Uma alteração não está concluída só porque compilou.

Para dizer **VALIDADA**, exigir as evidências aplicáveis:

- parser sem erro;
- runtime sem erro;
- expected == actual;
- marker `_OK` legítimo;
- arquivo persistido correto;
- regressões relacionadas passando;
- nenhum arquivo inesperado;
- warnings relevantes explicados;
- save/load preservado quando tocado;
- escopo autorizado respeitado.

---

# 23. Relatório final obrigatório após alterações

Sempre informar:

1. arquivos analisados;
2. arquivos alterados;
3. autorização usada;
4. mudança realizada;
5. output real do Godot;
6. testes executados;
7. regressões;
8. warnings/errors;
9. arquivos inesperados;
10. riscos restantes;
11. se o usuário ainda precisa validar manualmente.

---

# 24. Evidência histórica do benchmark

Os 27 testes que originaram esta skill estão documentados em:

```text
references/benchmark-27-testes.md
```

Consultar essa referência quando:

- houver dúvida sobre suporte de `Vector2`/`Color`;
- MCP reportar sucesso suspeito;
- for preciso justificar verificação pós-write;
- for desenhar stress test;
- for trabalhar com round-trip/save;
- for comparar com os três bugs reais encontrados;
- for explicar por que feature experimental não deve virar production automaticamente.

---

# 25. Regra final

> **O MCP é uma ferramenta de operação; o Godot real e o estado persistido são a evidência. A autorização do usuário controla a escrita.**

Em caso de dúvida entre “continuar alterando” e “parar para pedir autorização”, **parar e pedir autorização**.
