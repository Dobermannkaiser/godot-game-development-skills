# Design system, layout e responsividade

## Sumário

1. Tokens e `Theme`
2. Componentes e estados
3. Tipografia, ícones e densidade
4. Containers, anchors e tamanho
5. Resoluções, escala e breakpoints
6. Scroll, clipping e safe areas
7. Conteúdo extremo e localização
8. Fronteiras de dados e fidelidade de assets
9. Revisão estrutural

## 1. Tokens e `Theme`

Centralizar decisões recorrentes em tokens semânticos:

- superfícies e elevação;
- texto primário, secundário e desativado;
- ação, seleção, foco, sucesso, aviso e perigo;
- tipografia e alturas de linha;
- espaçamento;
- tamanhos mínimos de alvo;
- bordas, raios e sombras;
- duração e curva de movimento;
- opacidade;
- camadas e sons.

Preferir `color_text_warning` a `yellow_3`. O nome descreve o papel; o valor pode mudar.

Escala simples de espaçamento, adaptada ao projeto:

```text
4, 8, 12, 16, 24, 32, 48
```

No Godot:

- manter um `Theme` global ou por grande domínio visual;
- definir fonte padrão e tamanhos no tema;
- usar `StyleBox`, cores, constantes e ícones compartilhados;
- criar `theme_type_variation` semântica;
- evitar overrides locais repetidos;
- documentar exceções deliberadas;
- testar herança quando a árvore de `Control` muda.

Variações úteis:

```text
PrimaryButton
SecondaryButton
DangerButton
QuietButton
HeadingLabel
BodyLabel
CaptionLabel
PanelSurface
PanelRaised
ModalPanel
WarningBanner
```

Conferir a versão real do Godot antes de depender de classes, propriedades ou containers recentes.

## 2. Componentes e estados

Criar cenas reutilizáveis para padrões reais, não para cada combinação estética:

- cabeçalho de tela;
- botão com ícone/rótulo;
- ação perigosa;
- linha de configuração;
- cartão;
- item de lista;
- aviso/banner;
- tooltip;
- toast;
- modal;
- estado vazio/erro;
- barra de progresso;
- seletor;
- painel expansível.

API de componente deve receber dados de apresentação e emitir intenção. Não deve calcular economia, relacionamento, save ou condição narrativa.

Estados interativos relevantes:

- padrão;
- hover;
- pressionado;
- foco;
- selecionado;
- desativado;
- carregando;
- sucesso;
- aviso;
- erro.

Nem todo componente precisa desenhar todos os estados, mas o contrato deve definir os aplicáveis. Não depender de alteração sutil de cor:

- foco: contorno + mudança de superfície;
- selecionado: marca/forma + cor + rótulo quando necessário;
- desativado: aparência + motivo acessível;
- erro: borda/ícone + texto;
- carregando: estado ocupado + prevenção de duplicidade.

Não transformar toda diferença em componente novo. Preferir composição e variações semânticas.

## 3. Tipografia, ícones e densidade

Definir escala mínima:

```text
Título de tela
Título de seção
Corpo
Rótulo
Legenda
Número de destaque
```

Regras:

- usar fonte decorativa apenas onde continua legível;
- manter altura de linha e largura confortáveis em texto longo;
- evitar parágrafos em maiúsculas;
- não rasterizar texto necessário em imagens;
- testar números grandes, sinais, casas decimais e unidades;
- possuir fallback para idiomas e símbolos necessários;
- considerar fonte para dislexia como opção apenas se testada, não como solução universal;
- permitir escala de UI quando o jogo é denso em leitura.

Ícones:

- usar metáfora reconhecível ou rótulo;
- manter linguagem visual consistente;
- não comunicar status apenas por cor;
- adaptar ícone direcional em RTL quando semanticamente necessário;
- verificar legibilidade em escala menor e alto contraste;
- marcar elemento decorativo para não interferir com ponteiro ou semântica acessível.

Densidade pode variar por contexto ou preferência, mas não deve esconder informação crítica nem reduzir alvos abaixo do uso confortável.

## 4. Containers, anchors e tamanho

Preferir `Container` para conteúdo dinâmico:

- `VBoxContainer` e `HBoxContainer`;
- `GridContainer`;
- `MarginContainer`;
- `CenterContainer`;
- `PanelContainer`;
- `ScrollContainer`;
- `TabContainer`;
- `SplitContainer`;
- `FlowContainer`;
- outros compatíveis com a versão-alvo.

Usar coordenadas manuais para composição deliberada, HUD específico, elementos de mundo ou desenho procedural; não para formulários e listas com texto variável.

Anchors relacionam o controle ao pai; offsets refinam. Padrões:

- raiz: Full Rect;
- barra superior: esquerda/direita/topo;
- rodapé: esquerda/direita/base;
- painel lateral: topo/base e lado;
- modal: centro com limite mínimo/máximo e margem segura.

Verificar:

- `custom_minimum_size` necessário, não inflado;
- size flags coerentes;
- separação entre tamanho do conteúdo e do viewport;
- ordem de layout após mudanças deferred;
- nós ocultos que ainda reservam espaço conforme o container;
- `custom_maximum_size` somente quando suportado pela versão-alvo.

Evitar ler tamanho definitivo no mesmo instante em que altera a árvore/layout; aguardar a atualização apropriada quando a API exigir.

## 5. Resoluções, escala e breakpoints

Definir explicitamente:

- resolução base;
- menor viewport suportado;
- proporções alvo;
- modo de stretch;
- política de janela redimensionável;
- safe areas;
- escala de UI;
- limites de densidade.

Testar pelo menos:

```text
base
menor 16:9
maior 16:9
ultrawide
4:3 ou proporção estreita aplicável
janela redimensionada
UI 125%
UI 150%
texto expandido
```

Em plataformas onde o Godot não obtém escala do sistema de forma confiável, oferecer controle manual de escala quando necessário. Não inferir conforto físico apenas pela resolução.

Usar breakpoints pelo espaço disponível, não pelo nome do dispositivo. Adaptar estrutura:

- três colunas → duas → uma;
- painel lateral → aba/drawer;
- tabela larga → cartões ou detalhe;
- botões horizontais → fluxo vertical;
- cabeçalho completo → resumo compacto.

Não somente reduzir fonte, padding e alvo até caber. Preservar prioridade, toque e leitura.

Centralizar cálculo responsivo e torná-lo idempotente. Evitar que cada componente invente breakpoints conflitantes.

## 6. Scroll, clipping e safe areas

Estrutura robusta:

```text
PanelContainer
└── VBoxContainer
    ├── Header fixo
    ├── ScrollContainer (expandir)
    │   └── Content
    └── Footer fixo
```

Regras de scroll:

- rolar somente o conteúdo necessário;
- manter ação de saída acessível;
- garantir que foco selecionado seja trazido à área visível;
- testar roda, trackpad, teclado, controle e toque aplicável;
- definir comportamento de scroll aninhado;
- evitar saltos quando itens carregam;
- preservar posição somente quando ajuda a tarefa;
- não usar mínimo maior que o painel sem intenção.

Usar `clip_contents` quando visuais devem permanecer dentro do painel. Clipping não corrige layout incorreto.

Safe areas:

- reservar margem para recortes e barras de sistema;
- não colocar ação crítica na borda;
- respeitar áreas alcançáveis no toque;
- testar orientação e redimensionamento;
- não assumir que todo monitor exibe a área inteira sem overscan quando console/TV for alvo.

## 7. Conteúdo extremo e localização

Testar componentes com:

- título vazio e título muito longo;
- maior nome permitido;
- números negativos e muitos dígitos;
- lista vazia e lista muito grande;
- texto com quebra, emoji e caracteres combinantes;
- expansão de pseudolocalização;
- RTL aplicável;
- fonte substituta;
- escala alta;
- imagem ausente;
- dados parciais ou desatualizados.

Não fixar largura de botão ao texto original. Preferir wrap, mínimo, expansão, truncamento com acesso ao valor completo e reorganização.

## 8. Fronteiras de dados e fidelidade de assets

Componente reutilizável deve receber dados já validados ou normalizá-los numa fronteira única. Definir por campo:

- tipos aceitos;
- unidade e período;
- precisão e arredondamento;
- uso de sinal e separador local;
- estado para ausência, não aplicável e erro;
- comprimento máximo e comportamento responsivo.

Não chamar construtores como `String(valor)` sobre `Variant` arbitrário. Um valor de recurso pode chegar como `48.0` mesmo quando visualmente parece inteiro. Usar formatação explícita por tipo e fallback que não mascare defeitos. Testar `int`, `float`, negativo, zero, texto inesperado, `null` e valores não finitos quando aplicáveis.

Para retratos, expressões, ícones e texturas com transparência, validar no fundo real da interface:

- alpha e premultiplicação;
- `modulate`/`self_modulate` herdados;
- filtro, mipmaps e compressão de importação;
- escala, recorte e proporção;
- espaço de cor, contraste e contorno;
- mapeamento correto entre identidade, expressão e estado;
- placeholder de asset ausente sem trocar silenciosamente a identidade.

Uma imagem correta isoladamente pode parecer apagada, translúcida ou ilegível depois da importação e composição. Conferir a cena renderizada e comparar com a referência aprovada; análise do PNG sozinha não prova fidelidade.

## 9. Revisão estrutural

Checklist:

- [ ] Tema e tokens centralizados
- [ ] Sem overrides arbitrários repetidos
- [ ] Componentes não duplicam regras de gameplay
- [ ] Estados aplicáveis definidos
- [ ] Ação principal tem hierarquia
- [ ] Containers e anchors coerentes
- [ ] Mínimos não forçam overflow
- [ ] Layout estreito reorganiza
- [ ] Scale 150% continua funcional
- [ ] Conteúdo extremo não corta informação crítica
- [ ] Tipos, unidades, precisão e fallbacks definidos
- [ ] Zero, ausência, não aplicável e erro não se confundem
- [ ] Retratos/ícones mantêm alpha, identidade e legibilidade no fundo real
- [ ] Scroll mantém cabeçalho/saída e revela foco
- [ ] Safe areas respeitadas
- [ ] Decoração não bloqueia input
