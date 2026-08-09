# Godot: controles, input e foco

## Sumário

1. Arquitetura de camadas
2. Estado, sinais e ciclo de vida
3. Entrada de GUI
4. Mouse e toque
5. Foco e navegação
6. Modais e roteamento
7. Scroll, tooltips e popups
8. Dispositivo ativo e glifos
9. Padrões GDScript
10. Fronteira de apresentação e valores dinâmicos
11. Falhas recorrentes

## 1. Arquitetura de camadas

Estrutura conceitual:

```text
MainUI (CanvasLayer)
└── Root (Control, full rect)
    ├── ScreenLayer
    ├── HudLayer
    ├── OverlayLayer
    ├── ModalLayer
    ├── ToastLayer
    └── DebugLayer
```

Adaptar à arquitetura existente. O objetivo é separar responsabilidades, ordem visual e prioridade de entrada; não impor nomes.

Evitar usar apenas `z_index` para resolver tudo. Conferir:

- posição na árvore;
- `CanvasLayer`;
- raiz que recebe entrada;
- `mouse_filter` e comportamento recursivo disponível na versão;
- foco atual;
- visibilidade/processamento;
- subwindows e popups nativos.

Elevar a raiz funcional inteira de um modal, não apenas o painel desenhado.

## 2. Estado, sinais e ciclo de vida

Fluxo preferido:

```text
estado/domínio → sinal ou snapshot → UI renderiza
UI → intenção → domínio valida → estado muda → UI atualiza
```

Manter:

- regras fora de botões e labels;
- uma fonte de verdade;
- atualização idempotente;
- conexões de sinais sem duplicidade;
- desconexão ou troca segura de modelo;
- nós opcionais explicitamente tipados/validados;
- continuação após `await` protegida contra tela fechada ou operação obsoleta.

Fazer a UI consumir snapshot ou método de consulta do domínio para valores atuais, metas, previsões e causas. Não reconstruir na tela multiplicadores de dificuldade, estação, capacidade ou elegibilidade. Após carregar save, deixar o domínio reaplicar regras derivadas e emitir o estado reconciliado antes de renderizar.

Não usar `_process()` para reescrever textos e listas estáveis. Atualizar por eventos como:

```text
resource_changed
selection_changed
settings_changed
locale_changed
viewport_resized
input_mode_changed
opportunity_state_changed
forecast_changed
```

Se uma tela mostra requisito pendente, conectar sua atualização ao evento que pode satisfazê-lo, além de abertura da tela e load. Reavaliar apenas no próximo checkpoint pode deixar uma mensagem obsoleta por dezenas de ciclos.

Ao abrir/fechar repetidamente, verificar se callbacks, timers, tweens e bindings foram duplicados.

## 3. Entrada de GUI

Usar:

- `_gui_input(event)` para interação do próprio `Control`;
- sinais do componente para ações padrão;
- `_unhandled_input(event)` para atalhos globais ainda não consumidos;
- InputMap para ações remapeáveis;
- `_shortcut_input(event)` quando apropriado à versão e ao fluxo.

Prioridade conceitual:

1. modal ativo;
2. controle focado;
3. tela ativa;
4. atalho global;
5. gameplay.

Não capturar `Esc`, setas, clique ou roda globalmente antes da GUI. Marcar eventos como tratados apenas quando a camada realmente os consumiu.

Evitar lógica baseada em scancode físico quando a intenção deve ser remapeável ou localizada.

## 4. Mouse e toque

`mouse_filter`:

- `STOP`: recebe e bloqueia propagação conforme o fluxo;
- `PASS`: recebe e permite continuidade apropriada;
- `IGNORE`: não participa do ponteiro.

Elementos decorativos sobre botões normalmente usam `IGNORE`. Conferir a propriedade de comportamento recursivo se a versão do Godot a oferecer e houver um subtree inteiro a desabilitar.

Para ações de alto risco, confirmar no soltar quando o controle padrão permite cancelar movendo para fora. Evitar executar em `button_down` sem necessidade.

Toque:

- não depender de hover;
- definir rolagem versus toque de seleção;
- evitar alvos muito próximos;
- considerar multi-touch e gesto do sistema;
- não emular mouse sem entender eventos duplicados;
- fornecer alternativa a drag-and-drop.

Não alternar freneticamente o modo de entrada por ruído mínimo de mouse ou analógico. Aplicar zona morta/limiar e considerar o último evento intencional.

## 5. Foco e navegação

Todo fluxo principal suportado por controle/teclado deve funcionar sem mouse.

Usar ações padrão de UI (`ui_up`, `ui_down`, `ui_left`, `ui_right`, `ui_focus_next`, `ui_focus_prev`, `ui_accept`, `ui_cancel`) com cuidado: não removê-las do InputMap nem substituí-las por ações de gameplay sem preservar navegação.

Definir foco:

- `FOCUS_ALL` para controles navegáveis por teclado/controle;
- `FOCUS_CLICK` apenas quando navegação não deve alcançá-los;
- `FOCUS_NONE` para decoração/não interativo;
- `FOCUS_ACCESSIBILITY` somente em versões que oferecem esse modo e quando o elemento precisa entrar no fluxo do leitor de tela sem participar da navegação comum.

Não assumir que todo `Control` deve receber foco. O excesso cria ruído e passos inúteis.

Foco visível precisa sobreviver a:

- fundo claro/escuro;
- estado selecionado;
- modo de alto contraste;
- animação reduzida;
- escala de UI;
- teclado e controle.

Ordem automática por proximidade falha em grades, colunas, rodapés e layouts que mudam. Definir `focus_neighbor_*`, `focus_next` e `focus_previous` onde necessário e recalcular após mudança responsiva.

Ao abrir tela/modal:

1. registrar foco anterior;
2. aguardar a árvore/layout ficar pronta;
3. focar a ação segura ou primeiro item útil;
4. manter o foco dentro do modal;
5. trazer o item focado ao viewport rolável;
6. restaurar foco anterior válido ao fechar;
7. usar fallback lógico se o controle anterior sumiu.

Nunca iniciar foco em `Apagar`, `Sair sem salvar` ou equivalente.

## 6. Modais e roteamento

Um roteador pode oferecer:

```text
open_screen(id, context)
replace_screen(id, context)
go_back()
open_modal(id, context)
close_modal(result)
```

Responsabilidades:

- evitar duplicatas;
- manter histórico coerente;
- passar contexto tipado/validado;
- controlar transição;
- restaurar foco;
- resolver Voltar;
- não deixar telas ocultas consumirem input.

Modal correto:

- aparece acima;
- bloqueia camadas inferiores quando necessário;
- marca semântica modal em APIs de acessibilidade suportadas;
- captura foco;
- tem título e ação de saída;
- responde a `ui_cancel` conforme risco;
- define clique fora;
- restaura foco;
- remove bloqueio e processamento ao fechar.

Evitar modal sobre modal. Quando inevitável, manter pilha explícita e restaurar o modal anterior, não a tela de fundo.

Voltar consistente:

```text
se há modal → fechar/cancelar o modal
senão se há tela no histórico → voltar
senão → comportamento da raiz (por exemplo, confirmação de saída)
```

Não permitir que a mesma entrada feche duas camadas no mesmo frame.

## 7. Scroll, tooltips e popups

Ao mover foco em `ScrollContainer`, garantir que o controle fique visível. Evitar animação de scroll que atrase o acesso ou entre em conflito com repetição do controle.

Tooltips padrão podem não servir para conteúdo complexo. Um sistema customizado deve:

- responder a mouse e foco;
- ter atraso e cancelamento previsíveis;
- permanecer enquanto usuário interage quando apropriado;
- respeitar safe area;
- não interceptar ponteiro sem intenção;
- desaparecer ao mudar contexto;
- expor texto à acessibilidade.

Popups e menus:

- definir foco inicial;
- suportar busca quando a lista é longa e a versão do Godot oferece isso, ou implementar alternativa compatível;
- manter seleção atual visível;
- fechar com `ui_cancel`;
- devolver foco ao invocador;
- não usar menu contextual como único caminho para ação essencial.

## 8. Dispositivo ativo e glifos

Modos usuais:

```gdscript
enum InputMode { KEYBOARD_MOUSE, GAMEPAD, TOUCH }
```

Atualizar glifos quando o modo intencional muda. Não misturar instruções de dispositivos diferentes na mesma tela, exceto em tela de remapeamento/ajuda que compare explicitamente.

Considerar:

- layouts Xbox/PlayStation/Nintendo/genérico;
- controle desconectado;
- alternância durante modal;
- remapeamento;
- teclado ABNT e outras distribuições;
- texto localizado para nomes de ações;
- múltiplos jogadores e dispositivo por jogador, se aplicável.

Glifo representa a ação atual, não um botão fixo codificado.

## 9. Padrões GDScript

### Atualização de modelo sem conexão duplicada

```gdscript
var _model: ResourceModel

func bind_model(model: ResourceModel) -> void:
    if is_instance_valid(_model) and _model.changed.is_connected(_refresh):
        _model.changed.disconnect(_refresh)
    _model = model
    if is_instance_valid(_model) and not _model.changed.is_connected(_refresh):
        _model.changed.connect(_refresh)
    _refresh()
```

Adaptar tipos e nulabilidade à versão/projeto. Não copiar exemplo sem conferir a classe real.

### Foco de modal com fallback

```gdscript
var _previous_focus: Control

func open_modal(safe_target: Control) -> void:
    _previous_focus = get_viewport().gui_get_focus_owner()
    show()
    safe_target.grab_focus.call_deferred()

func restore_focus(fallback: Control) -> void:
    var target := _previous_focus if is_instance_valid(_previous_focus) else fallback
    if is_instance_valid(target) and target.is_visible_in_tree():
        target.grab_focus.call_deferred()
```

Captura de foco completa pode exigir lógica adicional no roteador. Não usar somente este fragmento para modais empilhados.

### Layout responsivo idempotente

```gdscript
const NARROW_WIDTH := 900.0

func _ready() -> void:
    get_viewport().size_changed.connect(_apply_layout)
    _apply_layout()

func _apply_layout() -> void:
    var narrow := get_viewport_rect().size.x < NARROW_WIDTH
    sidebar.visible = not narrow
    compact_navigation.visible = narrow
    cards.columns = 1 if narrow else 2
```

O breakpoint é exemplo. Incluir escala de UI, texto e mínimo real na decisão do projeto.

## 10. Fronteira de apresentação e valores dinâmicos

Centralizar formatação por tipo e significado. Exemplo mínimo:

```gdscript
func format_value(value: Variant) -> String:
    match typeof(value):
        TYPE_INT:
            return str(int(value))
        TYPE_FLOAT:
            var number := float(value)
            if is_nan(number) or is_inf(number):
                return "—"
            return "%.1f" % number
        TYPE_STRING, TYPE_STRING_NAME:
            return str(value)
        _:
            return "—"
```

Adaptar separador decimal, unidade, sinal e precisão ao projeto. Para população, contagem ou dia, preferir inteiro; para recursos e taxas, usar a precisão definida. Não usar `String(value)` como conversão genérica de `Variant`: tipos numéricos como `float` podem falhar no runtime mesmo quando o script passou por verificação estrutural.

Separar valor bruto de texto final. Uma linha de meta pode receber, por exemplo:

```gdscript
{
    "current": 48.0,
    "target": 43.0,
    "unit": "food",
    "decimals": 1,
    "status": "met"
}
```

Validar o contrato antes de montar `48,0 / 43,0`. Se o dado for inválido, apresentar estado seguro, registrar contexto técnico e manter a tela utilizável quando possível; não substituir silenciosamente por zero.

## 11. Falhas recorrentes

### Modal visível, botão não funciona

Inspecionar raiz, camada, foco, `mouse_filter`, controle transparente acima, modal anterior e evento global. Corrigir a ordem visual e de entrada juntas.

### Ícone bloqueia botão

Definir decoração como `MOUSE_FILTER_IGNORE` e verificar filhos adicionais.

### Interface funciona só com mouse

Configurar `focus_mode`, vizinhos, foco inicial, `ui_accept`, `ui_cancel`, scroll e estilo de foco; testar o fluxo inteiro.

### Foco some após reconstruir lista

Preservar identidade lógica do item, reconstruir de forma controlada e focar equivalente/fallback depois do layout. Não guardar apenas referência a nó que será liberado.

### `Esc` fecha modal e tela

Consumir o evento na camada superior e impedir que o mesmo evento atravesse ou seja processado duas vezes.

### Scroll corta foco

Revelar o controle focado, revisar header/footer, mínimos e scroll aninhado.

### Alternância infinita de glifos

Filtrar movimento mínimo, distinguir mouse real de movimento programático e aplicar limiar no analógico.

### Tela abre no editor, mas falha com um save real

Reproduzir com os tipos e valores do snapshot real. Conferir conversões de `Variant`, campos ausentes, dados antigos, números negativos e caminhos de fallback. Abrir a tela no Godot real; parse e inspeção estática não executam construtores nem callbacks de renderização.

### Progresso mudou, mas a tela continua bloqueada

Conferir se o domínio reavaliou elegibilidade, se emitiu o evento correto e se a UI está inscrita sem conexão duplicada. Testar também load com estado pendente. Não resolver com polling por frame nem com cálculo paralelo dentro do painel.
