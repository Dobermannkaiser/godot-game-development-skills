# Integração técnica de UI em Godot

## Sumário

1. Limite desta referência
2. Fluxo de dados da UI
3. Containers, anchors e responsividade
4. Modais, camadas e mouse
5. Foco, teclado e controle
6. Scroll e listas dinâmicas
7. Estado visual e assincronia
8. Acessibilidade e localização
9. Performance e testes visuais
10. Checklist

## 1. Limite desta referência

Usar esta referência para integração programática: sinais, estado, input, foco, lifecycle, persistência e testes. Para arquitetura de informação, design system, ergonomia, pesquisa, hierarquia visual e revisão completa, usar também `godot-ux-ui-para-jogos`.

Não redesenhar tela aprovada ao adicionar lógica. Mudança funcional deve preservar layout/identidade salvo pedido ou problema comprovado.

## 2. Fluxo de dados da UI

Preferir fluxo previsível:

```text
estado consistente → ViewModel/dados de apresentação → renderização
ação do jogador → comando validado → sistema altera estado → sinal → renderização
```

A UI pode formatar, filtrar e ordenar para apresentação, mas não deve possuir a única implementação da regra. Botão desabilitado não substitui validação no sistema.

Tratar a passagem de `Dictionary`/`Variant` para texto como fronteira tipada. Converter `int` e `float` explicitamente e usar `str()` apenas como fallback seguro; `String(48.0)` pode falhar no runtime. Centralizar formatação por tipo de métrica para que população permaneça inteira e recursos aceitem decimais.

Para tela de metas/previsões, preparar um modelo de apresentação auditável:

```text
valor atual | projeção | meta | prazo | estado | premissas/riscos
```

O modelo deve vir do calculador de domínio. A UI escolhe texto, cor e ícone, mas não recalcula produção, conclusão de obra ou elegibilidade.

Evitar atualizar todos os controles em `_process()`. Reagir a sinais e atualizar apenas região afetada. Para tela complexa, centralizar `refresh(state_snapshot)` ou funções por seção e impedir refresh reentrante.

Ao conectar itens dinâmicos, vincular ID estável:

```gdscript
button.pressed.connect(_on_item_pressed.bind(item_id))
```

Não capturar índice de lista se ordenação/filtro pode mudar antes do clique.

## 3. Containers, anchors e responsividade

Preferir `Container` para layout e anchors para relação com o pai. Não misturar posicionamento manual e container no mesmo filho esperando ambos prevalecerem.

Testar:

- resolução mínima oficial;
- aspect ratios estreito e largo;
- escala de interface;
- texto maior/localização;
- modo janela e tela cheia;
- safe areas em mobile, se suportado.

Usar `custom_minimum_size` com parcimônia. Um mínimo maior que a viewport pode quebrar scroll ou empurrar botões.

Ao adicionar aba/lateral, medir espaço restante. Pode ser melhor mover ferramenta interna para Configurações/Debug do que comprimir gameplay.

## 4. Modais, camadas e mouse

Um modal precisa resolver desenho e entrada. Estrutura segura:

```text
ModalLayer/CanvasLayer
└── Blocker (full rect, mouse stop)
    └── CenterContainer
        └── Panel
```

Ao abrir:

- mostrar camada inteira;
- bloquear input abaixo;
- elevar no `CanvasLayer`/ordem adequada;
- guardar foco anterior;
- focar primeira ação válida;
- pausar gameplay quando a regra exigir;
- impedir dois modais incompatíveis.

Ao fechar:

- cancelar operações da janela quando necessário;
- esconder/desativar blocker;
- restaurar foco se ainda válido;
- liberar referências e callbacks transitórios;
- retomar gameplay de modo coerente.

Exemplo:

```gdscript
func open_modal() -> void:
	_previous_focus = get_viewport().gui_get_focus_owner()
	visible = true
	mouse_filter = Control.MOUSE_FILTER_STOP
	confirm_button.grab_focus()

func close_modal() -> void:
	visible = false
	mouse_filter = Control.MOUSE_FILTER_IGNORE
	if is_instance_valid(_previous_focus):
		_previous_focus.grab_focus()
```

Não elevar apenas o painel visual enquanto o contêiner que recebe input continua atrás. Janela invisível deve usar `MOUSE_FILTER_IGNORE` ou sair da árvore quando aplicável; não pode interceptar cliques.

## 5. Foco, teclado e controle

Definir vizinhança/focus mode para navegação previsível. Verificar:

- foco inicial;
- ciclo dentro do modal;
- retorno ao elemento anterior;
- ações `ui_accept`, `ui_cancel` e navegação;
- controle desconectado/reconectado;
- mouse e teclado alternados;
- nenhum foco em elemento oculto/desabilitado.

Todo modal/tutorial deve ter rota visível e ação de cancelamento quando o produto permitir. Não depender somente de `Esc`, pois controle/mobile precisam de alternativa.

Usar `_unhandled_input()` para atalhos de gameplay para que controles de texto e UI possam consumir primeiro. Chamar `accept_event()` em `Control` quando o evento não deve atravessar.

## 6. Scroll e listas dinâmicas

Padrão:

```text
VBoxContainer
├── Header (fixo)
├── ScrollContainer (expandir + preencher)
│   └── ContentContainer
└── Actions (fixas)
```

O filho do `ScrollContainer` precisa calcular tamanho corretamente. Evitar mínimo vertical maior que a área rolável sem intenção. Manter ações essenciais fora do scroll quando devem permanecer acessíveis.

Para listas grandes:

- medir antes de virtualizar;
- reutilizar componentes se criação for gargalo;
- desconectar/resetar todo estado ao reutilizar;
- preservar foco/seleção por ID, não por índice;
- evitar reconstrução completa a cada pequeno sinal.

Ao filtrar/ordenar, anunciar estado vazio e manter critério visível. Não remover seleção silenciosamente sem atualizar detalhes.

## 7. Estado visual e assincronia

Modelar estados da tela:

- carregando;
- pronta;
- vazia;
- erro recuperável;
- ação em andamento;
- confirmação;
- indisponível por regra.

Não usar apenas `visible` espalhado por callbacks. Uma função de estado reduz combinações impossíveis.

Abrir uma aba ou modal é um caminho de runtime que precisa de regressão própria. Testar reconstrução com inteiro, decimal, campo ausente recuperável, coleção vazia e save antigo. Um painel informativo não pode derrubar o jogo inteiro porque uma métrica chegou como `float`; usar fallback visual seguro e registrar estrutura inesperada para diagnóstico.

Em ação assíncrona:

1. impedir clique duplo;
2. mostrar progresso proporcional;
3. manter token/geração;
4. validar resultado;
5. ignorar resposta obsoleta;
6. recuperar botão/estado em sucesso e erro;
7. não atualizar cena já fechada.

Para ações destrutivas, confirmação deve nomear objeto e consequência. Para ações reversíveis, preferir undo/toast quando adequado.

## 8. Acessibilidade e localização

Na integração técnica:

- usar chaves de tradução, não texto como ID;
- permitir expansão de texto;
- não codificar cor como único indicador;
- expor estados com ícone/texto/tooltip quando necessário;
- respeitar escala de UI e tamanho de fonte;
- manter contraste/tema no design system;
- evitar animações obrigatórias excessivas e oferecer redução quando aplicável;
- remapear controles;
- não depender apenas de hover.

Separar formatação de número/data da regra. Confirmar pluralização e direção do texto para idiomas suportados. Testar pseudolocalização ou strings longas quando houver pipeline.

## 9. Performance e testes visuais

Bugs que exigem execução visual:

- texto cortado;
- controle fora da tela;
- modal visível sem receber input;
- foco preso/perdido;
- clipping incorreto;
- tooltip fora da viewport;
- escala errada de retrato/texture;
- animação atualizando estado antigo;
- overlay bloqueando gameplay;
- lista reconstruindo e perdendo seleção.
- aba que compila, mas falha somente ao formatar dados reais;
- previsão que abre em save novo e quebra em save antigo;
- contador que mostra ativações quando deveria mostrar ofertas apresentadas.

Roteiro curto por tela:

1. abrir por todos os caminhos;
2. navegar com mouse;
3. navegar só com teclado/controle;
4. testar resolução mínima e texto longo;
5. abrir modal sobre outros painéis;
6. executar ação válida/inválida;
7. fechar durante operação/animacão;
8. salvar, recarregar e voltar à tela;
9. verificar ausência de erros no debugger.

Quando o usuário fornecer uma captura e stack trace, preservar os valores reais como caso manual de regressão. Reabrir exatamente a tela, no mesmo tipo de save e com a mesma classe de valor antes de ampliar o roteiro.

Para captura automatizada, estabilizar seed, resolução e estado. Comparação visual ajuda regressão, mas não substitui teste de interação.

## 10. Checklist

- [ ] UI exibe estado, não contém a única regra.
- [ ] `Variant` numérico é formatado por tipo sem construtor inválido.
- [ ] Atual, projeção, meta e premissas permanecem distinguíveis.
- [ ] IDs, não índices transitórios, identificam itens.
- [ ] Containers e mínimos funcionam na resolução mínima.
- [ ] Modal bloqueia desenho e input corretamente.
- [ ] Foco funciona com teclado/controle e retorna ao fechar.
- [ ] Controle invisível não intercepta mouse.
- [ ] Scroll mantém ações essenciais acessíveis.
- [ ] Resposta assíncrona obsoleta é descartada.
- [ ] Tela abre com decimal, vazio recuperável e save antigo.
- [ ] Texto localizado pode crescer.
- [ ] Estados não dependem apenas de cor/hover.
- [ ] Roteiro visual cobre abertura, ação, fechamento e reload.
