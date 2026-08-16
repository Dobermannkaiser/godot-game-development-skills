# Relatório Técnico Consolidado — Benchmark Codex + Godot MCP

**Projeto:** Golem’s Mandate
**Benchmark:** 27 testes progressivos de MCP, Codex e Godot real
**Data de consolidação:** 16 de agosto de 2026
**Idioma:** pt-BR
**Objetivo deste documento:** preservar em um único arquivo todo o conhecimento operacional obtido no benchmark, para que outro chat, outra IA ou uma sessão futura consiga continuar o trabalho sem repetir os 27 testes.

---

## 1. Resumo executivo

O benchmark avaliou, progressivamente, três camadas diferentes:

1. **MCP Godot puro** — conexão, criação de cenas/nós, propriedades, assets, execução, debug, stress e comportamento de erro.
2. **Codex + acesso direto aos arquivos + MCP Godot** — edição de GDScript/TS CN, execução no Godot real, leitura de debug, diagnóstico e correção autônoma.
3. **Trabalho sobre o código real do Golem’s Mandate** — integração com sistemas existentes, modificações controladas, investigação autônoma de bugs, regressões, save/load integrado, auditoria e consolidação de uma nova versão.

### Resultado geral

- **27 testes concluídos.**
- **MCP conectado e funcional com Godot via Steam.**
- **Stress de 200 nodes:** 200/200 operações confirmadas no arquivo salvo, sem corrupção ou perda de conexão.
- **Três bugs reais do jogo descobertos e corrigidos autonomamente:**
  - `PopulationState.gd` — inconsistência de capacidade habitacional entre setter e import.
  - `StoryManager.gd` — contexto customizado aceito em runtime mas rejeitado no round-trip.
  - `BuildingManager.gd` — terceira casa confundida com nível final de edifício durante import da fila.
- **Versão consolidada ao final:** **Golem’s Mandate v3.11.6**.
- **Suite permanente criada:** 5 regressões + 1 integração de estado completo.
- **Round-trip integrado, JSON, triple round-trip e determinismo:** aprovados.
- **Cena principal:** executada pelo MCP sem parser/runtime errors ou warnings relevantes no fechamento do benchmark.

### Conclusão principal

A combinação mais produtiva não é “MCP sozinho”. A configuração efetiva é:

```text
                    CODEX
                      │
        ┌─────────────┴─────────────┐
        ▼                           ▼
  acesso direto aos arquivos      MCP Godot
        │                           │
        ├─ GDScript                 ├─ executar Godot
        ├─ .tscn                    ├─ capturar debug
        ├─ referências              ├─ abrir editor
        ├─ diff/hash                └─ parar execução
        └─ edição
                      │
                      ▼
                 GODOT REAL
```

O **acesso direto aos arquivos** deve ser preferido para código e propriedades complexas. O **MCP** é especialmente valioso como ponte para o motor real: abrir, executar, coletar debug e confirmar comportamento.

---

## 2. Contexto técnico da configuração

### 2.1. Projeto usado no benchmark

Caminho local durante os testes:

```text
C:\Users\Paulo\Documents\golems-mandate-parte-3-etapa-12-v-3.11.5-(1)
```

O nome da pasta permaneceu antigo, mas ao final do Teste 27 o projeto interno foi promovido para:

```text
config/name="Golem's Mandate"
config/version="3.11.6"
run/main_scene="res://scenes/main.tscn"
```

Godot detectado na consolidação:

```text
4.7.1.stable.steam.a13da4feb
```

Formato de save preservado:

```text
SAVE_VERSION = 18
SAVE_SCHEMA_ID = "golems_mandate_part3"
```

Não houve mudança de schema apenas por causa da promoção 3.11.5 → 3.11.6.

### 2.2. Configuração MCP confirmada

Servidor:

```text
@coding-solo/godot-mcp
```

Configuração usada:

```text
command: npx
args:
  - -y
  - @coding-solo/godot-mcp
```

Variável de ambiente:

```text
GODOT_PATH=C:\Program Files (x86)\Steam\steamapps\common\Godot Engine\godot.windows.opt.tools.64.exe
```

O `cwd` do MCP ficou vazio. O projeto é escolhido pelo workspace/pasta aberta no Codex, e **não** substituindo `GODOT_PATH` pelo caminho do projeto.

### 2.3. Ferramentas MCP observadas

No início do benchmark o servidor expôs:

- `get_project_info`
- `list_projects`
- `get_godot_version`
- `launch_editor`
- `run_project`
- `stop_project`
- `get_debug_output`
- `create_scene`
- `add_node`
- `load_sprite`
- `save_scene`
- `export_mesh_library`
- `get_uid`
- `update_project_uids`

Não havia ferramentas dedicadas para ler/editar GDScript. Esse trabalho foi feito pelo acesso direto do Codex ao sistema de arquivos.

---

# 3. Resultado detalhado dos 27 testes

## Testes 1–4 — conexão, projeto, editor, execução e debug

### Teste 1 — reconhecimento do projeto/MCP

**Objetivo:** confirmar a cadeia ChatGPT Desktop/Codex → MCP → Godot → projeto.

**Resultado:** **APROVADO.**

O Codex reconheceu `project.godot`, versão do projeto, main scene, cenas importantes, scripts e autoloads. O servidor MCP foi listado corretamente.

### Teste 2 — controle do editor

**Objetivo:** verificar controle básico do Godot via MCP.

**Resultado:** **APROVADO.**

`launch_editor` funcionou corretamente.

### Teste 3 — execução do projeto/cena

**Objetivo:** verificar se o MCP conseguia iniciar execução real no Godot.

**Resultado:** **APROVADO.**

`run_project` funcionou e mais tarde mostrou-se capaz de executar também as cenas de probe especificadas no benchmark.

### Teste 4 — debug e encerramento

**Objetivo:** capturar saída real do Godot e encerrar a execução.

**Resultado:** **APROVADO.**

`get_debug_output` e `stop_project` funcionaram de maneira confiável durante todo o benchmark.

**Conclusão dos Testes 1–4:** a cadeia operacional estava funcional de ponta a ponta.

---

## Teste 5 — criação de cena e limite do nome da raiz

**Objetivo:** criar uma cena nova somente pelo MCP.

Foi criada:

```text
tests/mcp_test.tscn
```

**Resultado:** criação e tipo da raiz funcionaram. O nome da raiz não pôde ser configurado pela API de `create_scene` e ficou automaticamente como `root`.

**Classificação:** **APROVADO COM LIMITAÇÃO.**

**Limitação #1:** `create_scene` permite escolher o tipo da raiz, mas não expõe nome da raiz.

---

## Teste 6 — hierarquia de nodes e Label

Estrutura construída somente pelo MCP:

```text
root (Node2D)
├── Container (Node2D)
│   ├── ChildA (Node2D)
│   └── ChildB (Node2D)
└── TestLabel (Label)
```

`TestLabel` recebeu texto e offsets simples.

**Resultado:** estrutura e propriedades persistiram exatamente no `.tscn`.

**Classificação:** **APROVADO.**

A verificação posterior do arquivo confirmou que o “success” do MCP correspondia ao estado realmente salvo neste caso.

---

## Teste 7 — propriedades simples versus `Vector2`

Foram avaliados tipos de propriedades através de `add_node.properties`.

### Tipos confirmados

- `String` — funciona.
- `bool` — funciona.
- `int` — funciona.
- `float` — funciona.

Exemplos persistidos corretamente:

```text
visible=false
z_index=7
rotation=0.5
text="Property Test"
```

### `Vector2`

Uma tentativa como:

```json
{"x":120,"y":80}
```

foi aceita pelo schema e o MCP relatou sucesso, mas a propriedade `position` não apareceu no `.tscn`; o node permaneceu na posição padrão `(0,0)`.

**Classificação:** **PARCIAL.**

**Limitação #2:** o MCP pode aceitar tipos complexos sem convertê-los/persisti-los.

---

## Teste 8 — asset real com `load_sprite`

Asset utilizado:

```text
res://assets/etapa5/empty_lot.png
```

O MCP recebeu um `Sprite2D` já existente e atribuiu corretamente a textura.

O `.tscn` confirmou:

```text
texture = ExtResource(...)
```

com referência real ao PNG.

**Resultado:** textura persistiu corretamente.

**Observação importante:** `load_sprite` **não cria** o `Sprite2D`; o node precisa existir antes.

A posição enviada como `Vector2` não persistiu, reafirmando a limitação do Teste 7.

**Classificação:** **APROVADO PARA TEXTURA / FALHA PARA VECTOR2.**

---

## Teste 9 — quatro representações de `Vector2`

Foram testadas quatro formas:

```text
A: {"x":10,"y":20}
B: [30,40]
C: "Vector2(50, 60)"
D: {"type":"Vector2","x":70,"y":80}
```

O MCP aceitou e relatou sucesso para todas.

Verificação do `.tscn`:

- nenhuma propriedade `position` foi persistida;
- todos os nodes permaneceram em `(0,0)`.

**Resultado:** **0/4 representações funcionaram.**

**Classificação:** **FALHA FUNCIONAL COM FALSO SUCESSO DO MCP.**

---

## Teste 10 — enum e `Color`

### Enum

`horizontal_alignment=1` em `Label` persistiu corretamente.

**Enum/int:** confiável.

### `Color`

Quatro formas diferentes foram enviadas:

```text
A: {"r":1,"g":0,"b":0,"a":1}
B: [0,1,0,1]
C: "Color(0, 0, 1, 1)"
D: {"type":"Color","r":1,"g":1,"b":0,"a":1}
```

Todas foram aceitas e reportadas como sucesso.

No arquivo salvo, todas se tornaram:

```text
Color(0, 0, 0, 1)
```

ou seja, preto opaco.

**Classificação:**

- enum/int: **APROVADO**;
- Color: **FALHA PERIGOSA**.

`Color` é mais perigoso que `Vector2`: em vez de simplesmente omitir a propriedade, pode persistir **um valor errado** enquanto informa sucesso.

---

## Teste 11 — 11 propriedades simples em uma chamada

Foi criado `StressLabel` com 11 propriedades simples/enum em uma única chamada:

```text
text="MCP Stress Test"
visible=false
z_index=17
rotation=0.375
offset_left=101.5
offset_top=82.25
offset_right=317.75
offset_bottom=149.5
horizontal_alignment=1
vertical_alignment=1
uppercase=true
```

**Resultado:** **11/11 persistiram corretamente.**

**Conclusão:** batching de propriedades primitivas/simples é confiável.

---

## Teste 12 — cena UI relativamente complexa

Cena criada somente pelo MCP:

```text
root (Control)
└── MainPanel (Panel)
    ├── TitleLabel (Label)
    ├── DescriptionLabel (Label)
    ├── PrimaryButton (Button)
    ├── SecondaryButton (Button)
    ├── OptionCheck (CheckBox)
    ├── ProgressProbe (ProgressBar)
    └── NestedContainer (Control)
        └── NestedLabel (Label)
```

Foram 10 nodes incluindo a raiz, 9 chamadas `add_node`, 6 tipos de node e 55 propriedades solicitadas.

Auditoria posterior:

- 10/10 nodes corretos;
- 55/55 propriedades efetivas corretas;
- 46 serializadas explicitamente;
- 9 omitidas por serem iguais aos defaults do Godot.

**Classificação:** **APROVADO.**

### Comportamento interno observado

`create_scene` criou e removeu temporariamente:

```text
res://godot_mcp_test_write.tmp
```

como teste interno de permissão de escrita. Não deixou resíduo.

---

## Teste 13 — operações inválidas e qualidade dos erros

Foram provocadas falhas deliberadas.

### Parent inexistente

O MCP relatou sucesso ao adicionar um filho em um parent inexistente, mas o node não foi persistido.

**Qualidade:** **PERIGOSA — falso sucesso.**

### Tipo de node inexistente

O MCP retornou erro claro e detalhado e nada foi salvo.

**Qualidade:** **EXCELENTE.**

### Cena inexistente

Erro claro: scene file inexistente.

**Qualidade:** **EXCELENTE.**

### Asset inexistente

Erro claro: texture inexistente.

**Qualidade:** **EXCELENTE.**

### `load_sprite` em node inexistente

O MCP informou “Sprite loaded successfully”, mas nada foi alterado.

**Qualidade:** **PERIGOSA — falso sucesso.**

### Propriedade inexistente

O node e as propriedades válidas foram salvos, mas a propriedade inválida foi ignorada silenciosamente. O MCP informou sucesso geral.

**Qualidade:** **PERIGOSA — sucesso parcial silencioso.**

### Regra resultante

> **Nunca considerar a mensagem de sucesso do MCP como prova suficiente.**

Após qualquer alteração relevante, confirmar arquivo persistido e/ou comportamento no Godot.

---

## Teste 14 — stress de 50 Labels

Cena:

```text
tests/mcp_stress_test.tscn
```

Resultados:

- 50 planejados;
- 50 chamadas com sucesso MCP;
- 50/50 confirmados no arquivo;
- nenhum timeout;
- nenhuma perda de STDIO;
- nenhuma corrupção;
- aproximadamente **5,27 s por Label** em média.

O `ShapedTextDataAdvanced` vazado no fechamento cresceu para 100 referências (aproximadamente 2 por Label) e `FontAdvanced` ficou em 1.

**Classificação:** **APROVADO, COM WARNING DE RID/DESEMPENHO.**

---

## Teste 15 — stress de 200 Labels

Cena:

```text
tests/mcp_stress_200_test.tscn
```

Resultados:

- 200 planejados;
- 200 chamadas;
- 200 sucessos MCP;
- 200/200 confirmados no `.tscn`;
- 0 duplicatas;
- 0 nodes ausentes;
- 0 corrupção;
- conexão preservada durante ~21 minutos.

Média global:

```text
~6,23 s/node
```

Por blocos de 50:

```text
001–050: ~5,40 s/node
051–100: ~5,93 s/node
101–150: ~6,73 s/node
151–200: ~6,85 s/node
```

A degradação do primeiro ao último bloco foi aproximadamente 27%, com tendência a estabilizar depois de 150.

`ShapedTextDataAdvanced` chegou a 400 referências no encerramento.

**Classificação:** **APROVADO.**

**Conclusão:** o MCP aguenta volume estrutural significativo, mas é lento para geração em massa. 500/1000 nodes teria baixo valor prático adicional.

---

# 4. Testes de autonomia Codex + MCP

## Teste 16 — ciclo autônomo com Parser Error

Arquivos criados em `tests/autonomy/`.

Foi inserido deliberadamente:

```gdscript
print("MCP_AUTONOMY_TEST_OK"
```

Primeira execução real:

```text
Debugger Break, Reason: 'Parser Error: Expected closing ")" after call arguments.'
res://tests/autonomy/AutonomyProbe.gd:4
```

O Codex:

1. criou código;
2. executou pelo MCP;
3. capturou o erro real;
4. diagnosticou a linha;
5. corrigiu o arquivo;
6. reexecutou;
7. observou `MCP_AUTONOMY_TEST_OK`;
8. encerrou o projeto.

**Resultado:** **CICLO AUTÔNOMO COMPLETO: SIM.**

---

## Teste 17 — bug lógico sem erro técnico

Cálculo correto:

```text
(base_value + bonus) * multiplier
(100 + 25) * 2 = 250
```

Bug inicial:

```gdscript
base_value + bonus * multiplier
```

Primeira saída real:

```text
AUTONOMY_LOGIC_FAILED expected=250 actual=150
```

O Godot não tinha Parser Error nem runtime error. O Codex precisou usar comportamento esperado versus observado.

Correção:

```gdscript
(base_value + bonus) * multiplier
```

Segunda saída:

```text
AUTONOMY_LOGIC_OK expected=250 actual=250
```

**Resultado:** **CICLO AUTÔNOMO DE BUG LÓGICO: SIM.**

---

## Teste 18 — bug lógico entre dois scripts

Arquivos principais:

```text
EconomyMath.gd
AutonomyMultifileProbe.gd
```

Bug:

```gdscript
return initial_stock + produced + consumed
```

Saída inicial:

```text
MULTIFILE_CASE_A_FAILED expected=150 actual=210
MULTIFILE_CASE_B_FAILED expected=85 actual=135
MULTIFILE_TEST_FAILED
```

O Probe detectava a falha, mas a causa estava em outro script.

Correção genérica:

```gdscript
return initial_stock + produced - consumed
```

Saída final:

```text
MULTIFILE_CASE_A_OK expected=150 actual=150
MULTIFILE_CASE_B_OK expected=85 actual=85
MULTIFILE_ALL_OK
```

Somente `EconomyMath.gd` foi modificado na correção.

**Resultado:** **CICLO AUTÔNOMO MULTIARQUIVO: SIM.**

---

## Teste 19 — correção sem regressão

Função compartilhada possuía bug somente para uma condição específica:

```gdscript
if efficiency > 1.0:
    applied_efficiency = ceil(efficiency)
```

Primeira execução:

```text
REGRESSION_CASE_A_OK
REGRESSION_CASE_B_OK
REGRESSION_CASE_C_OK
REGRESSION_CASE_D_FAILED expected=225 actual=300
REGRESSION_TEST_FAILED
```

Estado antes:

```text
3/4 corretos
```

Correção genérica:

```gdscript
return (initial_amount + produced - consumed) * efficiency
```

Estado depois:

```text
4/4 corretos
0 regressões
REGRESSION_ALL_OK
```

**Resultado:** **CICLO AUTÔNOMO COM REGRESSÃO: SIM.**

---

# 5. Integração com código real do Golem’s Mandate

## Teste 20 — usar um sistema real sem modificá-lo

Sistema escolhido autonomamente:

```text
VillageDialogueManager
res://scripts/dialogue/DialogueManager.gd
```

Motivos da escolha:

- `RefCounted`;
- determinístico;
- estado apenas em memória;
- sem save obrigatório;
- sem RNG;
- sem dependência da main scene.

APIs reais usadas:

- `start_conversation()`
- `advance()`
- `choose()`
- `get_current_node()`
- `get_current_choices()`
- `get_history()`
- `is_active()`

Dois casos reais foram criados: conversa linear e conversa com escolha.

Primeira execução já passou:

```text
REAL_INTEGRATION_CASE_A_OK
REAL_INTEGRATION_CASE_B_OK
REAL_INTEGRATION_ALL_OK
```

Nenhum arquivo original foi alterado.

**Resultado:** **INTEGRAÇÃO AUTÔNOMA COM SISTEMA REAL: SIM.**

---

## Teste 21 — primeira alteração real de production code

Foi adicionada experimentalmente a `DialogueManager.gd`:

```gdscript
func try_choose(choice_id: String) -> bool:
    var result: Dictionary = choose(choice_id)
    return bool(result.get("chosen", false))
```

O benchmark estabeleceu baseline antes da alteração, verificou call sites, criou testes para escolha inválida, escolha válida e API antiga, e executou regressão.

Resultado:

```text
REAL_CHANGE_INVALID_CHOICE_OK
REAL_CHANGE_VALID_CHOICE_OK
REAL_CHANGE_OLD_API_OK
REAL_CHANGE_ALL_OK
```

Testes antigos também continuaram passando.

**Resultado:** **ALTERAÇÃO AUTÔNOMA REAL VALIDADA: SIM.**

**Importante:** essa API era uma **feature experimental de benchmark**, não uma necessidade aprovada do jogo. Foi removida no Teste 27.

---

## Teste 22 — integração de dois sistemas reais

Sistemas:

- `DialogueManager.gd`
- `NpcRelationshipManager.gd`

O Codex analisou a arquitetura e escolheu **não acoplar diretamente** os dois managers. Criou uma bridge experimental:

```text
DialogueRelationshipCoordinator.gd
```

Também adicionou APIs auxiliares experimentais ao manager de relacionamento.

Casos reais validados:

```text
MULTISYSTEM_POSITIVE_OK
MULTISYSTEM_NEGATIVE_OK
MULTISYSTEM_NEUTRAL_OK
MULTISYSTEM_INVALID_OK
MULTISYSTEM_ALL_OK
```

Estado demonstrado no Caso A:

```text
ANTES:
dialogue_node=opening
relationship=5
history_size=1
active=true

DEPOIS:
dialogue_node=reply
relationship=25
history_size=3
active=true
```

Também foi criado probe de regressão do fluxo de relacionamento.

**Resultado:** **INTEGRAÇÃO AUTÔNOMA DE DOIS SISTEMAS REAIS: SIM.**

**Importante:** coordinator e APIs auxiliares foram considerados experimentais e removidos no Teste 27. `NpcRelationshipManager.gd` foi restaurado byte a byte à baseline externa.

---

# 6. Descoberta autônoma de bugs reais

## Teste 23 — encontrar um bug sem receber o bug

O Codex recebeu apenas a tarefa de procurar uma inconsistência real e reproduzível.

Candidatos principais:

1. `PopulationState.gd` — setter de capacidade aceitava estado que import rejeitava.
2. `CalendarState.gd` — hipótese sobre `free_play_unlocked`.
3. `EventManager.gd` — estado parcial em `set_external_events()`.

Candidato escolhido: `PopulationState.gd`.

### Reprodução antes da correção

```text
BUG_HUNT_INPUT requested_capacity=11 setter_accepted=true exported_capacity=11
BUG_HUNT_EXPECTED=true
BUG_HUNT_ACTUAL=false
BUG_HUNT_REPRODUCED
```

Problema:

- `set_housing_capacity()` aceitava `11`;
- exportava `11`;
- `import_save_data()` rejeitava o mesmo valor por exigir múltiplo de 5.

O projeto também continha:

```text
HOUSING_CAPACITY_PER_HOUSE = 5
```

em `BuildingManager`.

### Correção

`set_housing_capacity()` passou a rejeitar capacidade que não fosse múltipla de cinco:

```gdscript
new_capacity % 5 != 0
```

### Validação posterior

```text
BUG_HUNT_ORIGINAL_CASE_OK
BUG_HUNT_EXTRA_CASE_A_OK
BUG_HUNT_EXTRA_CASE_B_OK
BUG_HUNT_BOUNDARY_OK
BUG_HUNT_ALL_OK
```

Cena principal também iniciou sem erros relevantes.

**Confiança declarada:** 98%.

**Resultado:** **BUG REAL DESCOBERTO E CORRIGIDO AUTONOMAMENTE: SIM.**

Esse fix foi preservado na v3.11.6.

---

## Teste 24 — diagnóstico a partir de relato humano vago

Relato fornecido ao Codex, em essência:

> alguns valores parecem aceitar uma configuração, mas ela não sobrevive quando o estado é reconstruído; verificar se existe outro caso dessa família.

O Codex traduziu o relato para estabilidade de round-trip e mapeou vários sistemas de persistência.

Candidatos principais:

1. `StoryManager`
2. `BuildingManager`
3. `EventManager`
4. `CalendarState`
5. `CampaignManager`

Foram executados candidatos de `StoryManager`, `EventManager` e `CalendarState`.

Resultados:

```text
StoryManager   → REPRODUZIDO
EventManager   → REJEITADO
CalendarState  → REJEITADO
```

### Bug real encontrado

Fluxo:

```text
queue_custom_dialogue("...", "seasonal_custom_notice")
→ aceita
→ export_save_data preserva
→ nova instância
→ import_save_data encontra whitelist fixa
→ rejeita
```

A API pública era genérica, o export preservava o contexto e `finish_dialogue()` já possuía tratamento padrão para contextos customizados.

### Correção

Em `StoryManager.gd`:

- ID/contexto passaram a usar normalização com `strip_edges()`;
- a whitelist incompatível foi removida;
- permaneceu a validação estrutural: ambos vazios ou ambos preenchidos.

### Validação

```text
NATURAL_BUG_ORIGINAL_FIXED
NATURAL_BUG_NORMAL_OK
NATURAL_BUG_BOUNDARY_OK
NATURAL_BUG_INVALID_OK
NATURAL_BUG_DOUBLE_ROUNDTRIP_OK
NATURAL_BUG_ALL_OK
```

**Confiança declarada:** 97%.

**Resultado:** **SEGUNDA INCONSISTÊNCIA REAL DE ROUND-TRIP DESCOBERTA E CORRIGIDA: SIM.**

Esse fix foi preservado na v3.11.6.

---

## Teste 25 — round-trip integrado de múltiplos sistemas

Este foi o maior teste funcional do benchmark.

O save real foi mapeado e um estado semântico não trivial foi construído envolvendo, entre outros:

- perfil;
- população;
- calendário;
- NPC;
- relacionamento;
- história;
- campanha;
- construções;
- recursos.

### Bug integrado descoberto

Antes da correção:

```text
IMPORT_BUILDINGS_FAILED
INTEGRATED_BUILDINGS_FAILED
INTEGRATED_ROUNDTRIP_FAILED
```

Ordem válida antes do save:

```text
construction_orders = [obra_000001]
target_level = 3
is_housing = true
next_order_sequence = 2
```

Depois do import falho:

```text
construction_orders = []
next_order_sequence = 1
```

### Reprodução mínima

```text
INTEGRATED_BUG_EXPECTED=queued:true target_level:3 direct_import:true json_import:true orders_after:1
INTEGRATED_BUG_ACTUAL=queued:true target_level:3 before_orders:1 direct_import:false direct_after:0 json_import:false json_after:0
INTEGRATED_BUG_REPRODUCED
```

A falha acontecia também sem JSON.

### Causa

`_sanitize_loaded_order()` tratava qualquer `target_level == FINAL_LEVEL` como edifício final que precisava de variante.

Para housing, porém, `target_level=3` significava **terceira casa**, não nível 3 de uma construção comum.

### Correção

A validação de variante final passou a excluir housing:

```gdscript
if (
    not is_housing
    and int(order.get("target_level", 1))
    == BUILDING_VARIANT_CATALOG_SCRIPT.FINAL_LEVEL
):
```

### Resultado integrado após a correção

Todos os importadores e invariantes passaram.

Hash semântico:

```text
BEFORE = a366b2e30b483d072aa7a88026651910ec0cb80edc94e568fbd2b331eef356a2
AFTER  = a366b2e30b483d072aa7a88026651910ec0cb80edc94e568fbd2b331eef356a2
```

Triple round-trip:

```text
A = B = C = a366b2e30b483d072aa7a88026651910ec0cb80edc94e568fbd2b331eef356a2
INTEGRATED_TRIPLE_ROUNDTRIP_OK
```

Determinismo:

```text
IMPORT A == IMPORT B
INTEGRATED_DETERMINISM_OK
```

Regressões anteriores também passaram.

**Resultado:** **ROUND-TRIP INTEGRADO REVELOU E CORRIGIU BUG REAL.**

Esse fix foi preservado na v3.11.6.

---

# 7. Auditoria e consolidação

## Teste 26 — auditoria forense pós-benchmark

Objetivo: classificar tudo que havia sido criado/modificado sem tocar nos arquivos.

### Descoberta importante

A pasta de trabalho **não continha `.git`**. Portanto não havia baseline Git local confiável.

A auditoria reconstruiu a origem provável das mudanças usando:

- conteúdo atual;
- timestamps;
- hashes registrados;
- nomes de testes;
- call sites;
- histórico dos benchmarks.

Confiança aproximada da atribuição: 95%.

### Classificação final

**Bug fixes reais a manter:**

- `PopulationState.gd`
- `StoryManager.gd`
- `BuildingManager.gd`

**Features experimentais a remover/revisar:**

- `DialogueManager.try_choose()`
- `NpcRelationshipManager.get_pair_score()`
- `NpcRelationshipManager.adjust_pair_score()`
- `_apply_pair_delta()` introduzido pelo benchmark
- `DialogueRelationshipCoordinator.gd`

**Testes permanentes sugeridos:**

- regressão PopulationState;
- regressão StoryManager;
- regressão BuildingManager;
- DialogueManager básico;
- fluxo oficial de NpcRelationshipManager;
- round-trip integrado.

### Integridade da auditoria

```text
FILES_CHANGED_BY_TEST_26 = 0
```

**Resultado:** **AUDITORIA PÓS-BENCHMARK CONCLUÍDA: SIM.**

---

## Teste 27 — consolidação real da v3.11.6

O projeto atual foi transformado de laboratório em nova versão consolidada.

Foi localizada uma baseline externa segura na Área de Trabalho e usada somente para leitura/comparação:

```text
C:\Users\Paulo\Desktop\GolemsMandate-Parte3-Etapa12-v3.11.5 (2)
```

### Features experimentais removidas

- `DialogueManager.try_choose()`
- `NpcRelationshipManager.get_pair_score()`
- `NpcRelationshipManager.adjust_pair_score()`
- `_apply_pair_delta()` experimental
- `DialogueRelationshipCoordinator.gd`

Busca global final: nenhuma ocorrência restante dessas features experimentais.

### Restauração do NpcRelationshipManager

`NpcRelationshipManager.gd` foi restaurado semanticamente e **byte a byte** à baseline externa.

SHA-256 final/baseline:

```text
83297496CEA2463D1CCADDF3A53F3E0A9E6864379789BCCC9A8DABC170941531
```

### Três fixes reais preservados

- `PopulationState.gd`: `new_capacity % 5 != 0`.
- `StoryManager.gd`: round-trip genérico de ID/contexto sem whitelist incompatível.
- `BuildingManager.gd`: variante final somente quando `not is_housing`.

Os três arquivos mantiveram os hashes registrados antes do Teste 27.

### Testes artificiais removidos

Foram removidas as suites artificiais e de benchmark, incluindo:

- `autonomy*`
- `real_change`
- `real_multisystem`
- probes históricos substituídos por regressões positivas
- cenas MCP de propriedades/UI
- stress 50/200

### Suite permanente final

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

Estado final da pasta `tests/`:

```text
13 arquivos
6 scripts
6 cenas
1 UID
0 referências res:// quebradas
0 scripts de teste órfãos
```

### Resultados das regressões permanentes

```text
POPULATION_ROUNDTRIP_OK
STORY_ROUNDTRIP_OK
BUILDING_QUEUE_ROUNDTRIP_OK
DIALOGUE_MANAGER_BASIC_OK
NPC_RELATIONSHIP_OFFICIAL_FLOW_OK
```

Todos com:

```text
errors = []
finalErrors = []
```

### Integração final

```text
INTEGRATED_ROUNDTRIP_ALL_OK
INTEGRATED_TEST_ALL_OK
INTEGRATED_TRIPLE_ROUNDTRIP_OK
INTEGRATED_DETERMINISM_OK
```

Hash semântico final:

```text
BEFORE = a366b2e30b483d072aa7a88026651910ec0cb80edc94e568fbd2b331eef356a2
AFTER  = a366b2e30b483d072aa7a88026651910ec0cb80edc94e568fbd2b331eef356a2
```

### Cena principal

Execução final:

```text
Parser errors finais: 0
Runtime errors finais: 0
Warnings relevantes finais: 0
errors = []
finalErrors = []
```

### Versão promovida

```text
config/version="3.11.6"
```

Identificadores internos também foram atualizados para `3.11.6`, mantendo:

```text
SAVE_VERSION = 18
SAVE_SCHEMA_ID = "golems_mandate_part3"
```

**Resultado:** **GOLEM'S MANDATE V3.11.6 CONSOLIDADA E VALIDADA: SIM.**

### Observação de rigor

A validação registrada neste benchmark é técnica/automatizada via Codex + MCP + Godot real. Este relatório não registra uma sessão manual de gameplay humano posterior à consolidação; não afirmar que essa validação humana ocorreu sem confirmação específica do usuário.

---

# 8. Matriz de confiabilidade do MCP observada empiricamente

## 8.1. Operações consideradas confiáveis quando usadas corretamente

| Operação/tipo | Resultado observado |
|---|---|
| `get_project_info` / versão | Confiável |
| `launch_editor` | Confiável |
| `run_project` | Confiável |
| `get_debug_output` | Confiável |
| `stop_project` | Confiável |
| `create_scene` | Confiável para criar cena e tipo da raiz |
| `add_node` com parent válido | Confiável para estrutura |
| `save_scene` | Confiável |
| String | Confiável |
| bool | Confiável |
| int | Confiável |
| float | Confiável |
| enum representado por int | Confiável |
| batch de propriedades simples | 11/11 no teste |
| `load_sprite` em Sprite2D existente | Textura persistiu corretamente |
| 50 nodes sequenciais | 50/50 persistidos |
| 200 nodes sequenciais | 200/200 persistidos |

## 8.2. Operações/tipos que exigem cautela forte

| Situação | Comportamento observado | Regra operacional |
|---|---|---|
| `Vector2` em `properties` | MCP relata sucesso, valor não persiste | Não confiar; editar `.tscn`/GDScript diretamente |
| `Color` em `properties` | MCP relata sucesso, pode persistir preto incorreto | Não usar genericamente; editar arquivo diretamente |
| parent inexistente | Falso sucesso possível | Verificar hierarquia persistida |
| `load_sprite` em node inexistente | Falso sucesso possível | Confirmar node + textura no `.tscn` |
| propriedade inexistente | Node pode salvar, propriedade é ignorada | Auditar propriedades efetivas |
| nome da raiz em `create_scene` | Não configurável | Editar arquivo diretamente se necessário |
| geração massiva | Funciona, mas fica progressivamente mais lenta | Preferir geração direta + validação quando apropriado |
| nodes de texto/visual | RID leak warnings no encerramento do processo de teste | Não confundir warning de teardown com sucesso funcional; investigar se ocorrer em runtime normal |

## 8.3. Regra de ouro

> **“MCP success” não é critério de aceitação.**

Uma alteração só deve ser marcada como válida quando houver evidência independente, por exemplo:

1. conteúdo persistido no `.tscn`/`.gd`;
2. debug real do Godot;
3. marker explícito de teste;
4. regressão relacionada;
5. ausência de parser/runtime errors relevantes.

---

# 9. Padrões de falso sucesso identificados

O benchmark encontrou três padrões que precisam permanecer documentados:

### 9.1. Sucesso sem persistência

Exemplo: `Vector2`.

```text
MCP → success
arquivo → propriedade ausente
runtime → valor default
```

### 9.2. Sucesso com valor incorreto

Exemplo: `Color`.

```text
MCP → success
pedido → vermelho/verde/azul/amarelo
arquivo → Color(0,0,0,1)
```

### 9.3. Sucesso parcial

Exemplo: propriedade inválida.

```text
node criado → sim
propriedades válidas → sim
propriedade inválida → ignorada
MCP → sucesso geral
```

Esses três comportamentos justificam uma política obrigatória de pós-verificação.

---

# 10. Desempenho e stress

## 10.1. 50 Labels

- 100% de persistência.
- ~5,27 s/node.
- 0 timeouts.
- conexão estável.

## 10.2. 200 Labels

- 100% de persistência.
- ~6,23 s/node em média.
- ~20m45s para a sequência de 200 nodes.
- aproximadamente 27% de degradação entre primeiro e último bloco de 50.
- conexão STDIO permaneceu funcional.

## 10.3. Implicação prática

O MCP é suficientemente estável para mudanças estruturais moderadas, mas não deve ser usado como gerador de centenas de nodes quando o mesmo resultado puder ser produzido com segurança por edição direta de arquivo + validação no Godot.

---

# 11. RID/resource leak warnings observados

Durante criação/fechamento de cenas com texto e elementos visuais apareceram warnings de recursos como:

- `ShapedTextDataAdvanced`
- `FontAdvanced`
- em alguns casos textura/RendererDummy

Nos testes de stress o número de referências cresceu com a quantidade de Labels.

Até o encerramento do benchmark:

- não houve corrupção de cena;
- não houve falha de save;
- não houve perda de nodes;
- não houve evidência de que esses warnings de teardown representassem bug funcional do projeto.

**Regra:** registrar e distinguir warnings do processo MCP/teste de vazamentos observados no gameplay normal. Não ignorar se passarem a ocorrer durante runtime real do jogo.

---

# 12. O que o benchmark comprovou sobre autonomia

A sequência dos Testes 16–25 demonstrou, em ordem crescente:

1. corrigir erro de parser a partir do output real;
2. detectar bug lógico sem erro técnico;
3. rastrear causa entre múltiplos scripts;
4. corrigir sem regredir comportamentos que já funcionavam;
5. ler e usar um sistema real do jogo;
6. modificar production code de maneira aditiva e testada;
7. integrar dois sistemas reais escolhendo uma arquitetura de bridge;
8. encontrar autonomamente um bug real sem receber arquivo/função;
9. transformar relato humano vago em hipótese técnica e rejeitar falsos positivos;
10. encontrar bug emergente somente em round-trip integrado de múltiplos sistemas;
11. executar regressão, triple round-trip, determinismo e cena principal;
12. auditar e consolidar a pasta em uma versão limpa.

A evidência de maior valor veio dos **Testes 23–25**, porque eles encontraram problemas reais não fabricados pelo benchmark.

---

# 13. Três bugs reais preservados na v3.11.6

## 13.1. PopulationState — capacidade em múltiplos de 5

**Contrato anterior quebrado:** setter aceitava capacidade arbitrária que import rejeitava.

**Fix:** rejeitar capacidade não múltipla de 5.

**Regressão permanente:**

```text
tests/regression/population_housing_capacity_roundtrip/
```

Marker esperado:

```text
POPULATION_ROUNDTRIP_OK
```

## 13.2. StoryManager — contexto customizado no round-trip

**Contrato anterior quebrado:** API pública aceitava contexto customizado, export preservava, import tinha whitelist incompatível.

**Fix:** normalização e paridade ID/contexto, sem whitelist incompatível.

**Regressão permanente:**

```text
tests/regression/story_custom_dialogue_roundtrip/
```

Marker esperado:

```text
STORY_ROUNDTRIP_OK
```

## 13.3. BuildingManager — terceira casa versus nível final

**Contrato anterior quebrado:** housing com `target_level=3` era tratado como construção final e exigia variante inexistente.

**Fix:** validação de variante final somente quando `not is_housing`.

**Regressão permanente:**

```text
tests/regression/building_queue_roundtrip/
```

Marker esperado:

```text
BUILDING_QUEUE_ROUNDTRIP_OK
```

---

# 14. Suite permanente após o Teste 27

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

Para considerar uma entrega tecnicamente validada, os markers aplicáveis devem aparecer **junto com**:

```text
Parser errors = 0
Runtime errors = 0
errors = []
finalErrors = []
```

Warnings relevantes também devem ser zero ou explicitamente explicados.

---

# 15. Hashes finais importantes da v3.11.6

Ao término do Teste 27:

```text
project.godot
3F2C267FBF51020FE23D338DA221884D0212C5FF616480CD5D01EE6382FBA72C

DialogueManager.gd
E1C3AEA974AE7182D0B4BF6AA12BDA86A705D21248D80E36A87F049B6E2769C1

NpcRelationshipManager.gd
83297496CEA2463D1CCADDF3A53F3E0A9E6864379789BCCC9A8DABC170941531

PopulationState.gd
52EC651F257CF33E1D630E06765F54ACE831CC86AF29067CD78DB5F9908B2A65

StoryManager.gd
6C18AA089E5115C0D04A63F91BEC66598D7ED58C811E052D0390A099025F296E

BuildingManager.gd
D068AA4EA4571CF865288137227244AC356B240EBCB5ED857A3D8A3699F6BBF8
```

Esses hashes documentam o estado observado naquele momento. Mudanças futuras legítimas naturalmente os alterarão.

---

# 16. Diretrizes operacionais derivadas do benchmark

## 16.1. Separar análise de alteração

Antes de escrever:

1. inspecionar arquivos reais;
2. identificar call sites;
3. definir hipótese/objetivo;
4. listar escopo exato;
5. pedir autorização explícita para alterar.

Autorização para investigar não equivale a autorização para editar.

## 16.2. Autorização deve ser por escopo

Uma autorização deve dizer, no mínimo:

- quais arquivos/pastas podem mudar;
- que tipo de alteração será feita;
- quais testes poderão ser criados/executados;
- se production code está ou não autorizado.

Se durante a tarefa surgir necessidade de alterar outro arquivo de produção, **parar e pedir nova autorização**.

## 16.3. Reproduzir antes de corrigir bugs desconhecidos

Fluxo obrigatório:

```text
suspeita
→ reprodução em production code real
→ output do Godot
→ BUG REPRODUZIDO
→ diagnóstico
→ autorização de correção
→ patch mínimo
→ mesmo cenário novamente
→ casos adicionais
→ regressão
```

Não modificar production code apenas porque uma leitura estática “parece” mostrar um bug.

## 16.4. Não alterar expected para fazer teste passar

É proibido transformar falha em sucesso por:

- mudar valor esperado depois de observar o resultado;
- remover caso de teste;
- imprimir `_OK` artificialmente;
- hardcodar inputs do probe;
- colocar branch específico para o nome do teste.

## 16.5. Preferir arquivo direto para tipos complexos

Para `Vector2`, `Color`, transforms e outros Variants complexos:

- preferir edição direta de `.tscn`/`.gd`;
- depois abrir/executar com Godot;
- não depender de `add_node.properties` genérico.

## 16.6. MCP é um executor/observador, não a fonte final da verdade

Ordem de confiança:

```text
1. comportamento observado no Godot
2. arquivo persistido real
3. teste/regressão reproduzível
4. debug output
5. mensagem "success" da ferramenta MCP
```

A mensagem do MCP é a evidência mais fraca.

---

# 17. Fluxo recomendado para qualquer alteração futura

## Etapa A — inspeção somente leitura

- confirmar projeto e versão;
- ler `project.godot`;
- localizar scripts/cenas relevantes;
- mapear call sites;
- identificar saves/testes relacionados.

## Etapa B — plano

Informar:

- objetivo;
- arquivos pretendidos;
- risco;
- estratégia de teste;
- arquivos que **não** serão tocados.

## Etapa C — autorização explícita

Antes da primeira escrita, pedir autorização ao usuário.

## Etapa D — baseline

Quando o risco justificar:

- hash dos arquivos;
- output dos testes atuais;
- snapshot do estado relevante.

## Etapa E — patch mínimo

- menor escopo possível;
- sem refactor oportunista;
- sem limpar código fora do problema;
- preservar APIs existentes quando possível.

## Etapa F — verificação estática

- diff real;
- referências;
- `.tscn` persistido;
- propriedades efetivas.

## Etapa G — Godot real

Via MCP:

```text
run_project
get_debug_output
stop_project
```

Nunca inferir sucesso só porque não houve crash.

## Etapa H — regressão

Executar testes específicos do sistema alterado.

Mudanças de save/estado devem, quando aplicável, incluir:

- round-trip;
- JSON round-trip;
- borda válida;
- entrada inválida;
- double/triple round-trip;
- determinismo.

## Etapa I — integração

Para mudanças amplas, executar:

```text
tests/integration/game_state_roundtrip/
```

## Etapa J — main scene

Iniciar `res://scenes/main.tscn` sem interação destrutiva com save real.

## Etapa K — auditoria final

Relatar:

- arquivos modificados/criados/removidos;
- diff resumido;
- hashes quando relevantes;
- outputs reais;
- regressões;
- warnings;
- limitações.

## Etapa L — promoção

A IA **não promove sozinha** uma entrega para “base estável”. A aprovação final pertence ao usuário.

---

# 18. Diretriz específica para saves

Save/load provou ser uma área de alto valor para testes automatizados.

Ao modificar:

- população;
- calendário;
- construções;
- história;
- campanha;
- eventos;
- relacionamentos;
- perfil;
- conselho;
- oportunidades;

perguntar sempre:

1. o runtime pode criar esse estado?
2. `export_save_data()` o preserva?
3. uma nova instância aceita o próprio estado exportado?
4. `export → import → export` é semanticamente estável?
5. há campo derivado/transitório que deve ser excluído da comparação?
6. uma reconstrução repetida causa deriva?

O Teste 25 demonstrou que uma falha de save pode emergir somente quando múltiplos managers são reconstruídos na ordem real.

---

# 19. Diretriz específica para investigação autônoma

Uma IA pode procurar bugs autonomamente, mas precisa seguir disciplina adversarial.

Para cada suspeita:

- evidência estática;
- comportamento esperado objetivo;
- risco de falso positivo;
- reprodução real;
- argumento contra a própria hipótese.

Se a reprodução não falhar, rejeitar o candidato.

Os Testes 23 e 24 mostraram que **rejeitar falsos positivos é parte do sucesso**.

---

# 20. Diretriz específica para alterações de arquitetura

O Teste 22 demonstrou que uma arquitetura experimental pode funcionar e mesmo assim **não ser uma feature aprovada**.

Portanto:

- não confundir “passou nos testes” com “deve entrar no jogo”;
- separar benchmark/protótipo de requirement real;
- preferir baixo acoplamento;
- não adicionar camada permanente apenas para facilitar um teste;
- revisar call sites reais antes de manter APIs novas.

No Teste 27, `try_choose`, APIs experimentais de pair score e o coordinator foram removidos apesar de funcionarem tecnicamente.

---

# 21. O que foi removido na consolidação

Foram retiradas da versão oficial final:

- APIs experimentais do Teste 21;
- coordinator e APIs experimentais do Teste 22;
- testes sintéticos dos Testes 16–19;
- stress 50/200;
- cenas de investigação MCP;
- probes históricos substituídos por regressões positivas.

O objetivo foi transformar evidência de benchmark em uma suite pequena de proteção permanente do jogo.

---

# 22. Estado final do benchmark

## Capacidade comprovada

```text
MCP puro
├── reconhecer projeto                          ✅
├── abrir/controlar Godot                       ✅
├── executar e coletar debug                    ✅
├── criar cenas e hierarquias                   ✅
├── propriedades simples                        ✅
├── UI relativamente complexa                  ✅
├── asset Texture2D                             ✅
├── 200 nodes sequenciais                       ✅
└── stop confiável                              ✅

Limitações MCP
├── root name em create_scene                   ❌/limitado
├── Vector2 via generic properties              ❌
├── Color via generic properties                ❌
├── parent inexistente pode dar falso sucesso   ⚠️
├── load_sprite em node inexistente             ⚠️ falso sucesso
├── propriedade inválida                        ⚠️ sucesso parcial
├── bulk grande                                 ⚠️ lento
└── RID leaks em teardown de probes visuais     ⚠️

Codex + arquivos + MCP
├── corrigir parser error                       ✅
├── diagnosticar bug lógico                     ✅
├── rastrear múltiplos arquivos                 ✅
├── preservar regressões                        ✅
├── entender código real                        ✅
├── alterar production code                     ✅
├── integrar sistemas reais                     ✅
├── escolher arquitetura                        ✅
├── encontrar bug sem localização fornecida     ✅
├── interpretar relato humano vago              ✅
├── rejeitar falsos positivos                   ✅
├── testar round-trip integrado                 ✅
├── triple round-trip                           ✅
├── determinismo                                ✅
├── auditar escopo                              ✅
└── consolidar nova versão                      ✅
```

---

# 23. Recomendação final para outro chat/IA

Ao receber este documento, tratar as seguintes informações como o estado de referência do benchmark:

1. O MCP Godot funciona e é útil, mas **não é confiável como única fonte de verdade**.
2. Para alterações complexas, usar **Codex/acesso direto aos arquivos + MCP para execução e debug**.
3. **Sempre verificar persistência e/ou comportamento real após writes MCP.**
4. Não usar generic `properties` do MCP para `Vector2` ou `Color` sem uma nova validação específica que prove suporte diferente.
5. Antes de qualquer alteração no projeto, seguir o **gate de autorização explícita** definido na skill `godot-mcp-controle-validacao`.
6. Bugs desconhecidos devem ser reproduzidos antes da correção.
7. Mudanças em save/estado devem ser protegidas por round-trip e, quando relevantes, determinismo/triple round-trip.
8. A suite permanente da v3.11.6 deve ser usada como base de regressão.
9. Os três fixes reais dos Testes 23–25 fazem parte da v3.11.6 e não devem ser removidos sem evidência nova e autorização.
10. A validação humana de gameplay posterior à consolidação não está registrada neste benchmark; não inventá-la.

---

# 24. Identificação da skill derivada deste benchmark

Este relatório acompanha a skill:

```text
name: godot-mcp-controle-validacao
display_name: Godot MCP — Controle, Segurança e Validação
version: 1.0.0
```

A skill transforma as conclusões acima em regras operacionais obrigatórias: autorização prévia, classificação de risco, verificação pós-MCP, reprodução antes de correção, regressões e proteção contra falsos sucessos.

---

**Fim do relatório consolidado dos 27 testes.**
