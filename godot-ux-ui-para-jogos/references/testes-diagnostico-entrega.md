# Testes, diagnóstico e entrega

## Sumário

1. Estratégia de validação
2. Avaliação heurística
3. Teste de tarefa
4. Matrizes
5. Verificação estrutural e automação
6. Contratos de dados, previsão e persistência
7. Inspeção visual e regressão
8. Diagnóstico
9. Desempenho
10. Ferramentas internas
11. Auditoria e severidade
12. Checklists
13. Entrega

## 1. Estratégia de validação

Separar evidências:

- **estática:** arquivos, tipos, referências, tokens, strings;
- **compilador real:** parse/importação, warnings e APIs da versão usada;
- **runtime:** cena abre, estado atualiza, input responde, foco muda;
- **visual:** layout, contraste, clipping, movimento;
- **auditiva:** volumes, repetição, sincronização;
- **tecnologia assistiva:** leitor de tela, remapeamento, escala;
- **humana:** tarefa compreendida e concluída.

Cada camada responde perguntas diferentes. Não transformar parse bem-sucedido em prova de UX.

Antes de testar, definir:

- versão e cena;
- estado inicial/fixture;
- tarefa;
- input;
- resolução e escala;
- resultado esperado;
- evidência a registrar.

## 2. Avaliação heurística

Revisar cada fluxo contra:

1. estado visível;
2. linguagem do jogador;
3. saída/cancelamento;
4. consistência;
5. prevenção de erros;
6. reconhecimento;
7. eficiência;
8. minimalismo funcional;
9. recuperação;
10. ajuda.

Também verificar:

- foco e entrada;
- responsividade;
- acessibilidade;
- localização;
- desempenho percebido;
- confiança no save/configuração.

Registrar evidência observável. `Parece confuso` é fraco; `o custo está em outra aba e três participantes procuraram no HUD` é útil.

Avaliação heurística não substitui teste com jogadores.

## 3. Teste de tarefa

Dar meta, não sequência de cliques:

```text
Descubra quanto alimento será necessário na próxima avaliação e prepare a vila.
```

Observar:

- primeiro clique/ação;
- hesitação e leitura;
- retornos;
- erros e recuperação;
- dúvidas verbalizadas;
- tempo e sucesso;
- uso de ajuda;
- confiança ao confirmar.

Não orientar durante o teste, salvo protocolo de resgate definido. Distinguir problema de interface, falta de conteúdo, regra não compreendida e bug.

Com poucos participantes, usar achados como evidência qualitativa, não estatística universal.

## 4. Matrizes

### Resolução e conteúdo

```text
base
mínima
maior 16:9
ultrawide
4:3/estreita aplicável
UI 125%
UI 150%
pseudolocalização
RTL aplicável
conteúdo máximo
int/float/zero/negativo/ausente/inválido
```

### Entrada

```text
mouse
teclado
controle
toque aplicável
alternância em tempo real
controle desconectado
remapeamento
```

### Estado

```text
normal
vazio
carregando
erro
parcial/desatualizado
desativado
valores extremos
ação em andamento
modal
modal empilhado inevitável
perda e restauração de foco
```

### Acessibilidade

```text
contraste
sem cor
alto contraste
reduzir movimento
sem áudio
legendas ampliadas
sem mouse
leitor de tela suportado
tempo estendido
```

## 5. Verificação estrutural e automação

Um verificador pode detectar:

- referências quebradas;
- cenas/componentes ausentes;
- tema global e variações obrigatórias;
- overrides ou cores fora dos tokens;
- controles interativos sem foco quando deveriam tê-lo;
- decoração com `mouse_filter` incorreto;
- modal sem saída;
- tela sem Voltar;
- strings rígidas/não traduzíveis;
- conexão de sinal duplicada em padrões conhecidos;
- mínimos suspeitos em scroll;
- nomes acessíveis ausentes em controles customizados críticos;
- assets/fontes sem fallback esperado.
- padrões frágeis de conversão ou concatenação de `Variant` em texto.

Automação de runtime pode testar:

- abrir telas/estados por fixture;
- histórico do roteador;
- foco inicial e restauração;
- `ui_cancel`;
- ação bloqueada;
- atualização após sinal;
- rechecagem após ganho de requisito e após load;
- `int`, `float`, negativo, zero, `null`, campo ausente e dado inesperado na mesma tela;
- correspondência entre previsão exibida e resultado resolvido no mesmo snapshot;
- persistência de configuração;
- ausência de erro no depurador;
- capturas determinísticas quando o ambiente permite.

Não tratar snapshot visual como substituto de interação. Mudança legítima exige revisão da referência, não atualização automática cega. Análise estática não executa construtores, callbacks de renderização nem abertura de telas; o Godot real pode revelar warning ou erro de tipo que o verificador não encontrou.

## 6. Contratos de dados, previsão e persistência

Criar fixtures pequenas a partir de estados reais do jogador. Preservar no caso de regressão:

- versão, cena, dia, dificuldade, seed e versão do save;
- ação exata que abre ou atualiza a tela;
- snapshot mínimo dos dados;
- stack trace/warnings;
- captura e valor observado;
- resultado esperado e tolerância.

Para formatação, cobrir pelo menos:

```text
48 (int)
48.0 (float integral)
-43.7 (float negativo)
0
null/campo ausente
texto inesperado
NaN/infinito quando a origem permitir
```

Verificar unidade, sinal, casas decimais, separador local, leitura acessível e fallback. Uma regressão para `String(48.0)` deve falhar antes da entrega, e a tela precisa permanecer segura diante de dado inválido quando o produto puder se recuperar.

Para previsão, congelar um snapshot e comparar três resultados:

1. decomposição apresentada;
2. saldo final previsto;
3. resultado do domínio ao resolver o mesmo período sem eventos adicionais.

Falha nessa igualdade indica fonte duplicada, ordem diferente de modificadores ou regra desatualizada na UI. Testar também modificadores combinados, como produção e consumo sazonais atuando simultaneamente.

Para pendências e oportunidades, cobrir:

1. checkpoint elegível;
2. checkpoint bloqueado;
3. requisito satisfeito depois do checkpoint;
4. marco posterior enquanto o anterior está pendente;
5. salvar/carregar com backlog;
6. aceitar, recusar ou adiar conforme o contrato;
7. atualização do contador agregado em cada transição.

Comparar campanha nova e save antigo. Distinguir regra derivada reaplicada no load de dado histórico que não deve ser concedido retroativamente.

## 7. Inspeção visual e regressão

Manter capturas representativas:

- menu;
- HUD;
- configuração;
- modal;
- lista longa;
- resolução mínima;
- foco visível;
- alto contraste;
- texto expandido.

Comparar:

- recorte e overflow;
- deslocamento inesperado;
- hierarquia;
- fonte/fallback;
- estado selecionado/foco;
- transparência/contraste;
- alpha, modulação e fidelidade de retratos/expressões no fundo real;
- ordem de camadas;
- conteúdo ausente.

Captura não prova hover, foco real, áudio, scroll, animação ou leitor de tela. Registrar essas camadas separadamente.

## 8. Diagnóstico

Fluxo:

1. reproduzir com versão, cena, resolução, input, estado, seed e save aplicáveis;
2. capturar erro/log/warnings/stack/cena remota/foco atual e valores exatos;
3. reduzir ao menor fluxo;
4. inspecionar camada visual e de input;
5. seguir estado e sinais;
6. testar hipótese estreita;
7. corrigir causa;
8. retestar fluxo e vizinhos.

Perguntas úteis:

- o controle está visível, processando e dentro da árvore?
- outro nó intercepta o ponteiro?
- o foco está em nó oculto/liberado?
- a lista foi reconstruída?
- o mesmo sinal conectou duas vezes?
- o layout ainda não recalculou?
- o texto veio vazio por estado ou localização?
- o modal inferior ainda consome eventos?
- a configuração persistida é inválida?
- a API existe na versão usada?
- o tipo real é `int`, `float`, `String`, `null` ou outro `Variant`?
- a UI está mostrando regra ativa ou uma cópia desatualizada?
- o evento que satisfaz o requisito dispara rechecagem e renderização?

Não corrigir sintomas com `call_deferred()`, `z_index`, `MOUSE_FILTER_IGNORE`, `is_instance_valid()` ou fallback textual sem explicar por que o ciclo, a ordem ou o contrato de dados exige isso.

Transformar evidência do jogador em regressão proporcional. Uma captura com `−43,7 por dia` orienta teste de decomposição e balanceamento; uma stack em `_create_goal_row()` orienta fixture com os tipos numéricos reais. Não generalizar um caso sem separar fato, inferência e hipótese.

## 9. Desempenho

Medir com profiler e monitores antes de otimizar.

Primeiro reduzir:

- atualização por frame;
- recriação de listas/cenas;
- formatação e tradução repetidas;
- conexões duplicadas;
- tweens acumulados;
- carregamento síncrono desnecessário;
- texturas grandes em UI;
- shaders de blur/transparência custosos;
- layout recalculado em cascata.

Listas grandes:

- paginação;
- carregamento incremental;
- pooling/reciclagem;
- virtualização;
- cache de miniaturas controlado.

Preservar identidade, foco, seleção e semântica ao reciclar itens. Não otimizar tornando a interface inacessível.

Desempenho percebido:

- resposta imediata à intenção;
- indicador após atraso curto, evitando piscar em operação instantânea;
- progresso real quando mensurável;
- cancelamento quando seguro;
- não mentir com 100% antes de concluir.

## 10. Ferramentas internas

Um laboratório de UI pode:

- abrir qualquer tela;
- injetar estado normal/vazio/erro/máximo;
- alternar resolução e escala;
- pseudolocalizar;
- alternar RTL;
- exibir retângulos e vizinhos de foco;
- mostrar `mouse_filter`, camada e z-order;
- alternar input/glifos;
- reduzir movimento;
- habilitar alto contraste;
- forçar acessibilidade quando suportada;
- tirar capturas nomeadas.
- inspecionar tipo, origem, unidade e última atualização de valores dinâmicos;
- listar checkpoint, elegibilidade, pendência, oferta e conclusão separadamente;
- comparar previsão com resolução para o mesmo snapshot.

Isolar da campanha real. Não sobrescrever save/configuração do jogador. Marcar claramente como ferramenta de desenvolvimento e excluir da exportação quando apropriado. Consumir as mesmas regras e catálogos do jogo; qualquer bypass, concessão artificial ou limite removido precisa estar visível e não pode sustentar conclusão sobre balanceamento ou UX normal.

## 11. Auditoria e severidade

Registro:

```text
ID
Tela/fluxo
Tarefa afetada
Problema
Evidência
Princípio
Severidade
Frequência
Impacto
Solução recomendada
Status
Versão corrigida
Reteste
```

Escala:

- 0: não é problema;
- 1: cosmético;
- 2: atrito pequeno;
- 3: importante, causa erro ou exclusão relevante;
- 4: bloqueia tarefa, causa perda ou risco grave.

Priorizar 4 e 3, depois problemas frequentes de nível 2. Agrupar causas sistêmicas: corrigir um componente/tema/roteador pode resolver várias ocorrências.

## 12. Checklists

### Nova tela

- [ ] Tarefa e ação principal claras
- [ ] Informação crítica e consequência visíveis
- [ ] Voltar/Cancelar
- [ ] Normal, vazio, carregando, bloqueado e erro aplicáveis
- [ ] Ação destrutiva protegida
- [ ] Tema/componentes existentes reutilizados
- [ ] Resolução mínima e UI 150%
- [ ] Conteúdo longo e extremo
- [ ] Inteiro, decimal, negativo, zero, ausência e dado inválido
- [ ] Unidade, período, precisão e fallback coerentes
- [ ] Previsão usa a mesma fonte do runtime
- [ ] Eventos relevantes e load revalidam a tela
- [ ] Mouse, teclado, controle e toque aplicável
- [ ] Foco visível, ordem e restauração
- [ ] Strings traduzíveis
- [ ] Contraste, cor e movimento
- [ ] Sem erros no depurador

### Modal

- [ ] Raiz inteira acima
- [ ] Entrada inferior bloqueada
- [ ] Foco inicial seguro
- [ ] Foco preso na camada apropriada
- [ ] `ui_cancel` e clique fora definidos
- [ ] Foco anterior/fallback restaurado
- [ ] Sem bloqueio invisível ao fechar
- [ ] Nome/papel modal acessível quando suportado

### Scroll/lista

- [ ] Header/footer conforme necessidade
- [ ] Conteúdo interno rola
- [ ] Foco é revelado
- [ ] Roda, teclado, controle e toque aplicável
- [ ] Conteúdo máximo
- [ ] Estado vazio
- [ ] Identidade preservada após atualização
- [ ] Contadores distinguem futuro, bloqueado, pendente e concluído
- [ ] Virtualização não quebra seleção/semântica

### Acessibilidade

- [ ] Contraste medido
- [ ] Cor redundante
- [ ] UI/texto escaláveis
- [ ] Foco visível e não oculto
- [ ] Sem dependência exclusiva de hover/arrasto/som
- [ ] Reduzir movimento
- [ ] Legendas/TTS aplicáveis
- [ ] Remapeamento aplicável
- [ ] Nomes/estados para leitor de tela na versão suportada
- [ ] Linguagem simples

## 13. Entrega

Entregar:

- base e versão-alvo;
- problema do jogador e resultado;
- telas/componentes alterados;
- estados, entradas e resoluções cobertos;
- mudanças em tema/tokens;
- acessibilidade/localização;
- testes executados com resultado;
- evidência de compilação/runtime no Godot real ou ausência explícita;
- coerência entre previsão, Guia, ferramenta interna e regra ativa;
- limitações;
- roteiro manual priorizado;
- pendências.

Critérios de conclusão:

- tarefa compreensível;
- estado e feedback presentes;
- inputs alvo funcionam;
- foco lógico e visível;
- sem bloqueio invisível;
- resolução mínima, escala e texto extremo funcionam;
- estados de dados tratados;
- ações irreversíveis protegidas;
- configurações persistem;
- acessibilidade e localização revisadas;
- regressões verificadas;
- Godot real testado ou ausência declarada.

Não promover versão estável. O usuário testa e aprova explicitamente.
