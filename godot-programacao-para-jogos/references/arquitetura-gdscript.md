# Arquitetura e GDScript em Godot 4.x

## Sumário

1. Princípios de arquitetura
2. Organização de cenas e nós
3. Estado, regras e coordenação
4. Resources e conteúdo orientado a dados
5. Sinais e dependências
6. Tipagem e estilo de GDScript
7. Ciclo de vida, assincronia e memória
8. Input, tempo e determinismo
9. Autoloads, serviços e troca de cenas
10. Revisão rápida

## 1. Princípios de arquitetura

Projetar a partir dos fluxos reais do jogo, não de uma árvore de pastas idealizada. Separar quando houver motivos concretos:

- **estado:** dados mutáveis da campanha ou sessão;
- **regra:** cálculo e validação sem dependência visual;
- **coordenação:** ordem de operações e integração entre domínios;
- **apresentação:** cenas, animação, áudio e interface;
- **conteúdo:** definições imutáveis ou validadas;
- **infraestrutura:** save, configurações, logging e diagnóstico.

Uma organização possível, não obrigatória:

```text
scripts/
├── models/
├── systems/
├── services/
├── catalogs/
├── ui/
├── save/
├── diagnostics/
└── shared/
```

Preferir composição a heranças profundas. Usar herança para relação genuína de substituição e API estável; usar componentes/nós filhos/Resources para capacidades combináveis.

Não aplicar padrões por moda. Um arquivo grande pode merecer divisão quando possui múltiplos motivos de mudança; dez arquivos minúsculos acoplados podem ser piores.

## 2. Organização de cenas e nós

Tratar cenas como unidades reutilizáveis com contrato claro:

- raiz representa o conceito da cena;
- dependências obrigatórias são exportadas, criadas pela cena ou verificadas cedo;
- cenas filhas não procuram serviços atravessando muitos `..`;
- grupos representam categorias, não substituem referência direta quando só existe um destinatário;
- nomes únicos de cena (`%Nome`) podem reduzir caminhos frágeis, mas continuam sendo parte do contrato da cena;
- `owner` precisa estar correto quando nós criados por ferramenta devem ser salvos em `PackedScene`.

Antes de mover ou renomear nós, procurar:

- `NodePath` exportado;
- `%NomeUnico`;
- `$Caminho` e `get_node()`;
- animações que apontam para propriedades;
- conexões persistidas na cena;
- chamadas RPC dependentes do mesmo `NodePath`;
- testes e ferramentas de editor.

Não editar manualmente UID, cabeçalhos ou referências de `.tscn`/`.tres` sem entender o formato e conferir o diff. Preferir o editor quando a alteração depende de reimportação, ownership complexo ou serialização interna.

## 3. Estado, regras e coordenação

Modelos de estado devem:

- usar tipos e limites explícitos;
- ter defaults centralizados;
- expor serialização/deserialização defensiva;
- evitar referência direta a controles ou cenas;
- manter invariantes após qualquer operação pública.

Separar fatos persistidos de valores derivados. Progresso acumulado, escolhas, IDs materializados e itens pendentes são fatos; limites, duração configurada, capacidade calculada e metas vindas de catálogo normalmente devem ser recalculados. Se um valor derivado for salvo por conveniência, definir qual fonte vence após load e testar a mudança de regra entre versões.

Exemplo de regra pura:

```gdscript
class_name RelationshipRules
extends RefCounted

const MAX_POINTS: int = 1000

static func apply_delta(current: int, delta: int) -> int:
	return clampi(current + delta, 0, MAX_POINTS)
```

Coordenação deve tornar a ordem visível. Para uma passagem de dia:

```text
validar ação → calcular produção/consumo → aplicar eventos
→ resolver sequências obrigatórias → atualizar metas → emitir sinais → salvar
```

Quando uma ação envolve múltiplos domínios, preparar resultados antes de aplicá-los. Se não houver transação real, usar um objeto/Dictionary de resultado validado e só então efetivar a mudança.

## 4. Resources e conteúdo orientado a dados

Usar custom `Resource` quando for útil ter:

- edição no Inspector;
- tipos, propriedades exportadas e validação;
- referências entre assets;
- serialização nativa em `.tres`/`.res`;
- definições reutilizáveis.

Usar JSON/CSV quando conteúdo externo, pipeline de escrita ou interoperabilidade justificar. Validar esquema e converter para tipos internos na fronteira.

Resources carregados do mesmo caminho podem ser compartilhados. Se contiverem estado mutável por instância:

- duplicar de modo apropriado;
- considerar `resource_local_to_scene` quando o comportamento desejado for por cena;
- separar definição (`CharacterDefinition`) de estado (`CharacterState`);
- não modificar catálogo global durante gameplay.

Exemplo:

```gdscript
class_name BuildingDefinition
extends Resource

@export var id: StringName
@export var display_name: String
@export_range(1, 10) var max_level: int = 1
@export var costs_by_level: Array[Dictionary] = []
```

IDs persistentes devem ser estáveis, não localizados e nunca derivados do texto exibido. Validar duplicatas, referências ausentes, ciclos inválidos e dados incompatíveis na inicialização ou em ferramenta de diagnóstico.

Carregar recursos de projeto com `load()`/`preload()`/`ResourceLoader`, não com `FileAccess` bruto quando o export pode converter extensões ou empacotar assets.

## 5. Sinais e dependências

Usar sinais para fatos concluídos ou solicitações assíncronas bem nomeadas:

```gdscript
signal relationship_changed(character_id: StringName, previous: int, current: int)
signal dialogue_requested(dialogue_id: StringName)
```

Boas práticas:

- emitir depois que o estado estiver consistente;
- incluir dados suficientes para o observador reagir sem reler tudo;
- evitar sinal genérico `state_changed` para todos os domínios;
- impedir conexões duplicadas;
- desconectar quando a vida do emissor e observador não for naturalmente gerenciada pela árvore;
- não usar sinal para esconder dependência síncrona indispensável.
- emitir eventos semânticos que permitam reavaliar sistemas dependentes no momento correto, como `relationship_changed`, `building_completed` ou `difficulty_rules_applied`;
- reavaliar também após load quando o evento original já ocorreu, mas o estado dependente ainda pode estar pendente.

Conexão segura:

```gdscript
var callback: Callable = _on_button_pressed
if not button.pressed.is_connected(callback):
	button.pressed.connect(callback)
```

Preferir referência exportada ou injeção pelo criador da cena para dependência obrigatória. Usar grupos para broadcast/categorias; usar autoload para serviço realmente global; usar sinal para evento.

## 6. Tipagem e estilo de GDScript

Seguir o estilo do projeto e a versão mínima do motor. Preferir:

- tipos explícitos em APIs, estado persistente e coleções importantes;
- `StringName` para IDs/chaves comparados frequentemente e nomes de engine;
- `Array[Tipo]` quando o elemento é homogêneo;
- dicionários tipados apenas quando suportados pela versão mínima;
- enums ou constantes para estados fechados;
- retornos coerentes em todos os caminhos;
- conversão explícita ao ler `Variant`, JSON ou Dictionary.

Exemplo:

```gdscript
func add_points(character_id: StringName, amount: int) -> int:
	var current: int = int(_points.get(character_id, 0))
	var updated: int = clampi(current + amount, 0, 1000)
	_points[character_id] = updated
	return updated
```

Não confiar que métodos de `Array`/`Dictionary` sempre preservam tipo no retorno. Conferir a documentação da versão e converter/validar onde necessário.

Na fronteira com UI, logging e localização, formatar `Variant` pelo tipo real. `String(valor)` não é um conversor universal em GDScript 4.x e pode falhar com `float`. Usar `str(valor)` como fallback textual e formatação explícita quando a regra visual importa:

```gdscript
func format_metric(value: Variant, whole_number: bool = false) -> String:
	if value is int:
		return str(value)
	if value is float:
		return str(roundi(value)) if whole_number else "%.1f" % float(value)
	return str(value)
```

Testar a função com inteiro, `48.0`, decimal, `null` e tipo inesperado. A UI pode apresentar fallback seguro, mas deve registrar ou propagar estrutura essencial inválida em vez de ocultar corrupção.

Helpers internos devem ter nomes específicos e prefixo `_`. Evitar nomes que colidam com métodos virtuais, funções globais ou membros nativos de `Node`, `CanvasItem`, `Control` e outras bases. Não criar `_process`, `_draw`, `_input` ou callbacks semelhantes por acidente. Evitar também nomes como `seed` para variáveis locais quando o compilador reportar sombreamento de identificador global.

Não silenciar warnings de arquivo inteiro. Usar `@warning_ignore` no menor escopo e documentar por que o caso é seguro. Warnings podem estar configurados como erros. Tratar `SHADOWED_GLOBAL_IDENTIFIER`, `CONFUSABLE_LOCAL_DECLARATION`, parâmetro não usado e variável não usada como dívida real: renomear por significado, remover resíduo ou prefixar `_` somente quando a API precisa manter o parâmetro.

## 7. Ciclo de vida, assincronia e memória

Conhecer a ordem:

- `_init()`: objeto criado, árvore e filhos ainda podem não estar prontos;
- `_enter_tree()`: entrou na árvore;
- `_ready()`: filhos prontos;
- `_process()`: frame variável;
- `_physics_process()`: passo fixo;
- `_exit_tree()`: saída da árvore;
- `queue_free()`: liberação adiada com segurança.

Não acessar `@onready` em `_init()`. Evitar lógica importante dependente de ordem acidental entre irmãos.

Após `await`, o mundo pode ter mudado. Usar um token/geração quando a validade lógica importa:

```gdscript
var _request_generation: int = 0

func refresh_later() -> void:
	_request_generation += 1
	var generation: int = _request_generation
	await get_tree().process_frame
	if generation != _request_generation or not is_inside_tree():
		return
	_refresh_view()
```

`is_instance_valid()` evita acessar objeto liberado, mas não prova que a operação ainda é desejada. Verificar também cena atual, estado, token, visibilidade ou identidade da solicitação.

Usar `call_deferred()` para resolver exigência real de árvore/bloqueio de física, não como correção universal de ordem. Cancelar timers, tweens e callbacks quando necessário.

## 8. Input, tempo e determinismo

Definir ações no Input Map. Não codificar teclas físicas como única interface. Escolher callback pelo propósito:

- `_input`: recebe cedo;
- `_gui_input`: interação de `Control`;
- `_unhandled_input`: gameplay após UI não consumir;
- polling com `Input`: estados contínuos, quando necessário.

Consumir evento na camada correta para impedir clique/tecla atravessando modal.

Usar `_physics_process(delta)` para movimento/física determinística por passo e `_process(delta)` para visual. Não acumular lógica de calendário em FPS.

Para sorte testável:

```gdscript
var rng: RandomNumberGenerator = RandomNumberGenerator.new()

func set_simulation_seed(seed_value: int) -> void:
	rng.seed = seed_value
```

Salvar semente ou estado do RNG quando repetibilidade de campanha, replay, geração procedural ou depuração exigir. Separar RNG de conteúdo cosmético do RNG de gameplay para que uma animação não altere resultados. Quando vários sistemas de gameplay precisam ser reproduzíveis, derivar streams independentes por domínio ou persistir seu estado; a ordem de uma fala não deve mudar a economia ou um recrutamento.

## 9. Autoloads, serviços e troca de cenas

Autoload é adequado para serviços com vida de aplicação:

- roteamento de cenas;
- configurações globais;
- áudio persistente;
- save manager;
- telemetria/logging local, se existente.

Evitar estado de tela, referência a nós transitórios e regras de todos os domínios no mesmo singleton. Expor métodos pequenos e sinais tipados.

Na troca de cena:

- bloquear solicitações concorrentes;
- salvar somente em estado consistente;
- limpar overlays e input modal;
- preservar apenas serviços intencionalmente globais;
- garantir que callbacks antigos não atualizem a nova cena;
- tratar falha de carregamento sem deixar tela vazia.

Para carregamento interativo/assíncrono, acompanhar status e progresso, validar o recurso carregado e instanciar no thread principal.

## 10. Revisão rápida

- [ ] Estado não depende da UI.
- [ ] Cena possui contrato claro e caminhos estáveis.
- [ ] IDs são estáveis e validados.
- [ ] Resources mutáveis não vazam estado entre instâncias.
- [ ] Sinais representam fatos ou pedidos claros.
- [ ] Tipos são compatíveis com a versão mínima.
- [ ] `Variant` é validado e formatado com casos numéricos reais.
- [ ] Compilação real não acusa sombreamento, declaração confundível ou resíduos não usados.
- [ ] `await` e troca de cena não deixam callbacks obsoletos.
- [ ] Input respeita UI e remapeamento.
- [ ] RNG de gameplay é reproduzível quando necessário.
- [ ] Valores derivados possuem fonte de verdade definida após load.
- [ ] Autoloads permanecem pequenos e intencionais.
