# MCP Godot para UI, input, foco e validação

## Sumário

1. Divisão de responsabilidades
2. Governança e autorização
3. Limites do MCP em cenas e propriedades
4. Exclusividade entre interfaces
5. Navegação interna e foco
6. Estratégia de reprodução
7. Estados difíceis e dados sintéticos
8. Movimento reduzido e tweens
9. Evidência e linguagem correta
10. Checklist

## 1. Divisão de responsabilidades

Usar:

```text
edição direta → .gd, .tscn, Theme, InputMap e testes
MCP Godot     → reconhecer, executar, coletar debug e parar
Godot real    → lifecycle, foco, propagação e comportamento
inspeção visual/humana → legibilidade, sensação e qualidade final
```

MCP não substitui inspeção de cena, diff, runtime nem julgamento humano. Editar `Control`, foco e propriedades complexas diretamente quando a serialização for conhecida; usar editor visual quando ownership, anchors ou recursos serializados tornarem a edição textual arriscada.

## 2. Governança e autorização

Antes de alterar UI de produção:

1. inventariar interfaces/estados afetados;
2. reproduzir mouse, teclado e ações aplicáveis;
3. registrar expected/actual, foco atual e camada ativa;
4. localizar causa sistêmica e ocorrências;
5. propor solução mínima, arquivos e regressões;
6. aguardar autorização explícita.

Autorização para `tests/` não autoriza `scripts/ui/`, managers, cenas, themes, assets, `project.godot`, InputMap ou saves. Se a solução sistêmica exigir arquivo adicional, parar e pedir ampliação.

## 3. Limites do MCP em cenas e propriedades

Benchmark com Godot 4.7.1 confirmou que o servidor pode executar projetos, coletar output e criar hierarquias com propriedades simples. Também confirmou falsos sucessos:

- `Vector2` genérico pode não persistir;
- `Color` genérico pode persistir como preto;
- parent ou nó alvo inexistente pode ser ignorado apesar de `success`;
- propriedade inexistente pode ser ignorada;
- raiz de cena criada pode manter nome `root`.

Para UI isso afeta anchors, offsets, tamanhos, cores, modulate, posições e propriedades compostas. Preferir edição direta ou API especializada validada. Sempre reler `.tscn`/`.tres`, conferir valor exato, parent, owner e tipo, depois executar.

## 4. Exclusividade entre interfaces

Separar dois problemas:

- **exclusividade global:** qual interface/camada pode receber input;
- **navegação interna:** quais controles da interface ativa podem receber foco.

Um overlay visual não basta. Enquanto menu, modal ou diálogo estiver ativo:

- fundo não recebe mouse, teclado, controle ou shortcut incompatível;
- `Tab`, `Shift+Tab` e setas não escapam;
- `Accept` age somente na camada ativa;
- `Cancel` afeta apenas a camada apropriada;
- fechar restaura foco válido ou fallback seguro;
- nenhum blocker invisível permanece.

Investigar em conjunto `CanvasLayer`, visibilidade, processamento, `mouse_filter`, `_input`, `_shortcut_input`, `_unhandled_input`, `_gui_input`, InputMap, foco e roteador central.

Uma interface pode expor uma raiz de foco ativa equivalente a `active_focus_root` para subpágina, confirmação ou submodal. O gate global deve conhecer a camada realmente interativa, não apenas a janela exterior marcada como aberta.

## 5. Navegação interna e foco

Filtrar candidatos centralmente:

- visível na árvore;
- habilitado;
- `focus_mode` elegível;
- interativo;
- descendente da raiz ativa;
- não coberto por submodal prioritário.

Controles desabilitados não entram no ciclo. `Tab` percorre somente elegíveis. Setas devem respeitar neighbors válidos e usar fallback geométrico restrito à raiz ativa quando não houver candidato. Preservar esquerda/direita semânticos de abas quando o produto os utiliza.

Ao focar filho de `ScrollContainer`, revelá-lo automaticamente. Ao fechar, restaurar o controle anterior apenas se ele ainda estiver válido, visível e elegível; caso contrário usar fallback lógico.

Não permitir controle essencial mouse-only. Testar `ui_accept`, `ui_cancel`, navegação, foco inicial, reconstrução de lista, mudança responsiva e alternância entre dispositivos.

## 6. Estratégia de reprodução

Antes de executar numa auditoria declarada somente leitura, mapear `.godot/`, logs, `user://`, configurações, autosave e qualquer ferramenta que possa persistir estado. Preferir cópia temporária, diretório de usuário isolado e autosave/telemetria suspensos; comparar baseline e estado final dos destinos graváveis. Se não houver isolamento suficiente, limitar-se à inspeção estática e manter runtime/estados dependentes como `N/A`, ou pedir autorização para os efeitos inevitáveis.

Para auditoria global:

1. inventariar telas, subpáginas, modais e estados interativos;
2. marcar `PASS`, `FAIL` ou `N/A` com motivo;
3. para cada estado alcançável, abrir pelo fluxo real;
4. registrar foco inicial;
5. percorrer Tab/Shift+Tab e setas;
6. testar Accept/Cancel e mouse;
7. verificar tentativa de escapar para o fundo;
8. fechar e verificar restauração;
9. repetir após reconstrução e scroll.

Converter cada `FAIL` reproduzível em regressão. Não declarar que estados `N/A` passaram. Uma correção sistêmica deve retestar todos os FAIL e uma amostra de PASS para detectar regressão.

## 7. Estados difíceis e dados sintéticos

Menus dependentes de saves, dias futuros, ofertas ou decisões irreversíveis podem ser preparados com dados sintéticos somente em memória:

- usar produção real para renderização e regras;
- suspender autosave;
- não modificar save real;
- capturar/restaurar estado;
- tornar artificialidade explícita;
- não inferir balanceamento ou progressão normal do bypass.

Se o estado ainda não puder ser atingido, manter `N/A`. Não falsificar cobertura.

“Em memória” descreve o dado sintético, não garante ausência de escrita do motor. Auditar também save/configuração, project settings, recursos importados, caches e logs após o teste; releitura de `.tscn`/`.gd` sozinha não detecta esses efeitos colaterais.

## 8. Movimento reduzido e tweens

Reduzir duração para valor quase zero não é necessariamente movimento reduzido. Um tween de `0,01 s` com callback que reinicia pode criar ciclo contínuo, consumo de CPU e mudanças por frame.

Ao ativar redução de movimento:

- cancelar tweens ativos quando apropriado;
- bloquear callbacks tardios;
- posicionar elementos em estado válido e determinístico;
- não recriar tweens continuamente;
- restaurar comportamento normal ao desativar;
- não reiniciar trajetos por mudança de configuração não relacionada;
- verificar reconstrução, escala e troca de contexto.

Medir antes/depois com contagem de tweens distintos, tweens ativos e mudanças de posição no mesmo intervalo. Depois validar visualmente as posições em campanha real.

## 9. Evidência e linguagem correta

Distinguir:

- inspeção de arquivos;
- teste automatizado no Godot;
- eventos/actions sintéticos;
- mouse/teclado físicos usados pelo operador;
- controle/gamepad físico;
- validação visual humana;
- campanha completa.

Eventos do InputMap podem provar roteamento lógico de ações associadas ao gamepad; não dizer “testado com controle físico” sem dispositivo real. Captura prova aparência de um estado, não navegação. Parser limpo não prova input. Automação de foco não prova conforto.

## 10. Checklist

- [ ] Base estável, versão do Godot e escopo autorizados registrados.
- [ ] Interfaces e estados inventariados.
- [ ] Exclusividade global separada da navegação interna.
- [ ] Foco restrito à raiz realmente ativa.
- [ ] Invisíveis, desabilitados e `FOCUS_NONE` excluídos.
- [ ] Tab, setas, Accept e Cancel testados.
- [ ] Mouse e shortcuts não atravessam modais.
- [ ] Scroll revela o controle focado.
- [ ] Foco anterior ou fallback é restaurado.
- [ ] Movimento reduzido não cria loops de tween.
- [ ] Dados sintéticos não tocaram saves reais.
- [ ] Runtime somente leitura usou cópia/sessão isolada e auditoria antes/depois, ou permaneceu `N/A`.
- [ ] Writes MCP foram conferidos no arquivo persistido.
- [ ] `PASS`, `FAIL` e `N/A` não foram confundidos.
- [ ] Eventos sintéticos não foram descritos como dispositivo físico.
- [ ] Godot foi encerrado e validação humana pendente foi declarada.
