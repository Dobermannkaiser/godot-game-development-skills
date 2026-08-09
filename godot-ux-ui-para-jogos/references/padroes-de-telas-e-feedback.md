# Padrões de telas e feedback

## Sumário

1. Estados e feedback
2. Menu e criação de campanha
3. Configurações
4. Save e load
5. HUD e notificações
6. Jogos de gestão
7. Cartões, listas e detalhes
8. Diálogos e relacionamentos
9. Recomendações contextuais
10. Previsões, dificuldade e explicabilidade
11. Oportunidades e conteúdo procedural
12. Antipadrões

## 1. Estados e feedback

Toda interface conectada a dados deve definir os estados aplicáveis:

- inicial/carregando;
- carregado;
- vazio;
- conteúdo parcial;
- bloqueado/sem permissão;
- erro recuperável;
- falha crítica;
- ação em andamento;
- sucesso;
- dado desatualizado;
- item removido;
- conexão perdida, quando aplicável.

Em jogo offline, carregamento ainda existe para save, importação, assets, geração procedural e simulação.

Feedback proporcional:

- mudança local: atualização junto ao controle;
- conclusão comum: confirmação breve/toast;
- risco persistente: banner ou indicador durável;
- ação irreversível: confirmação específica;
- falha que bloqueia campanha: modal ou tela dedicada.

Não usar modal para toda informação. Não deixar tela branca ou botão congelado sem estado.

Feedback multimodal pode combinar visual, som, vibração e texto. Nenhum canal deve ser único para informação essencial.

## 2. Menu e criação de campanha

Menu principal:

- com save válido, destacar `Continuar`;
- sem save, destacar `Novo jogo`;
- manter `Carregar`, `Configurações`, `Acessibilidade/Guia` e `Sair` conforme plataforma;
- informar quando `Continuar` aponta para save incompatível/corrompido;
- proteger nova campanha se ela sobrescrever algo.

Criação de campanha:

- nome e opções necessárias;
- dificuldade com efeitos concretos;
- resumo antes de iniciar;
- validação de entrada;
- retorno sem perda;
- consequência sobre saves;
- defaults seguros;
- foco lógico.

Não usar dificuldade apenas como rótulo abstrato. Explicar mudanças como recursos, penalidades, ritmo e assistência.

## 3. Configurações

Categorias possíveis:

- Jogabilidade;
- Interface;
- Acessibilidade;
- Áudio;
- Vídeo;
- Controles;
- Idioma.

Regras:

- cabeçalho e Voltar acessíveis;
- conteúdo rolável sem perder foco;
- valor atual visível;
- restaurar padrões por categoria ou global com confirmação adequada;
- configurações globais separadas de estado da campanha;
- prévia reversível para escala, contraste e áudio;
- mudanças de vídeo arriscadas com contagem e reversão automática;
- remapeamento com conflitos e saída segura;
- persistência validada.

Ao mudar idioma ou escala, preservar a tela e o item em foco quando possível. Evitar que aplicar configuração feche a tela inesperadamente.

## 4. Save e load

Slot pode mostrar:

- nome;
- data/hora localizada;
- tempo jogado;
- local/etapa/dia;
- dificuldade;
- versão do jogo/save;
- miniatura quando agrega reconhecimento;
- estado de compatibilidade.

Estados:

- vazio;
- válido;
- migrável;
- incompatível futuro;
- corrompido com backup recuperável;
- salvando;
- falha.

Regras:

- diferenciar `Salvar` de `Carregar` visual e verbalmente;
- confirmar sobrescrita com slot nomeado;
- não ocultar falha de persistência;
- preservar campanha aberta quando save falha;
- impedir ação duplicada enquanto grava;
- manter backup/restauração como comportamento do sistema, explicado quando necessário;
- não prometer compatibilidade que a programação não garante.

## 5. HUD e notificações

Mostrar apenas o necessário ao contexto atual:

- sempre visível;
- aparece quando há risco;
- aparece sob consulta;
- aparece em modo específico.

O HUD deve responder:

- onde estou/qual modo;
- o que mudou;
- qual ameaça é urgente;
- qual ação está disponível;
- qual objetivo está próximo.

Notificações:

- priorizar e agrupar;
- limitar repetição;
- permitir histórico;
- não cobrir ações críticas;
- pausar ou adaptar em diálogo/cutscene;
- não usar mesma intensidade para tudo;
- oferecer redução de movimento e som.

Toasts não servem para informação que o jogador precisa consultar depois.

## 6. Jogos de gestão

Uma tela de gestão deve facilitar comparação e previsão. Mostrar, conforme a mecânica:

- valor atual;
- variação e período;
- produção/consumo;
- capacidade;
- meta;
- prazo;
- previsão;
- principais causas;
- decomposição entre base, produção, consumo e modificadores relevantes;
- consequência;
- ação recomendada opcional.

Exemplo:

```text
Alimentação: 84
Saldo previsto amanhã: -12
Meta: 70 em 4 dias
Situação prevista: segura
```

Não apresentar falsa precisão. Indicar incerteza, intervalo ou pressupostos quando eventos podem alterar a previsão. Se o saldo for extremo, mostrar as causas antes de recomendar ação; `−43,7 por dia` isolado não explica se o problema vem de consumo-base, estação, dificuldade, evento ou erro.

Para filas:

- posição e progresso;
- custo já pago;
- início e conclusão previstos;
- capacidade/limite;
- efeito de reordenar;
- regra e valor de cancelamento;
- bloqueio explicado;
- foco e seleção preservados após mudança.

Para retornos decrescentes/sinergias:

- mostrar regra e efeito efetivo;
- não esconder penalidade;
- exibir bônus baixo sem exagero visual;
- manter cálculo no domínio e somente apresentar na UI.

## 7. Cartões, listas e detalhes

Cartão deve favorecer leitura rápida e comparação. Definir:

- identidade;
- estado principal;
- valores essenciais;
- ação;
- selecionado/focado/desativado;
- conteúdo expandido;
- texto máximo;
- leitura por leitor de tela.

Para conteúdo procedural, definir também campos obrigatórios, diversidade mínima, seed/versão do gerador para diagnóstico e semântica de dado ausente. Não usar `Sem profissão`, retrato genérico ou zero como fallback silencioso para uma especialização que deveria ter sido gerada.

Em listas densas:

- cabeçalho/colunas claros;
- ordenação com direção indicada;
- filtros ativos visíveis e removíveis;
- contagem de resultados;
- estado vazio específico;
- seleção persistente por ID lógico;
- item focado revelado;
- virtualização sem perder ordem semântica.

Quando virtualizar, não depender do nó visual como identidade do item. Reutilização deve atualizar todo estado: texto, ícone, seleção, foco acessível, tooltip, sinais e estilo.

Detalhe expandido deve evitar repetir tudo. Usar para histórico, causas, requisitos, estatísticas e ações raras.

## 8. Diálogos e relacionamentos

Diálogos:

- nome/retrato consistentes;
- texto legível e localizado;
- histórico acessível;
- indicador de avanço;
- controle de velocidade/auto quando aplicável;
- avançar por mouse, teclado e controle;
- escolhas com texto completo;
- fundo sonoro reduzido quando necessário;
- legendas e identificação do locutor.

Não embaralhar respostas quando posição carrega significado de navegação. Não criar padrão visual que revele opção correta sem intenção de design.

Escolhas irreversíveis devem comunicar consequência no nível apropriado ao jogo, sem necessariamente revelar spoiler. A confirmação pode existir para romance, aliança ou ruptura se clique acidental tiver alto custo.

Relacionamentos podem mostrar:

- nível/marco;
- progresso e limite;
- estado de vínculo;
- eventos disponíveis;
- bônus descobertos;
- motivo de bloqueio;
- histórico narrativo relevante.

Não reduzir relação a barra abstrata sem contexto de eventos e decisões.

Após uma resposta ou acontecimento, atualizar pontos e marco somente depois da confirmação do domínio. Se a alteração falhar, preservar a escolha narrativa e explicar o que não foi aplicado conforme a política do jogo. Auditar todas as variantes procedurais e localizadas contra limites de conteúdo sensível definidos pelo projeto; um caminho alternativo não pode reintroduzir tema removido.

## 9. Recomendações contextuais

Separar motor de recomendação da apresentação:

```text
estado → regras de sugestão → fila priorizada → componente → resposta do jogador
```

Contrato conceitual:

```gdscript
{
    "id": "food_risk",
    "priority": 80,
    "title": "Prepare-se para o inverno",
    "message": "O estoque cobre 6 dias no ritmo previsto.",
    "action_label": "Ver produção",
    "target": "economy/food",
    "dismissible": true,
    "reason": "days_remaining < days_until_winter"
}
```

Categorias:

- crítica;
- importante;
- recomendação;
- descoberta;
- informativa.

Recomendação deve explicar por quê. Permitir aceitar, ignorar ou silenciar conforme risco. Não executar a decisão em nome do jogador.

## 10. Previsões, dificuldade e explicabilidade

Tratar previsão como resultado de domínio apresentado, não como cálculo independente da tela. Para o mesmo snapshot, alinhar:

```text
regra/catálogo → previsão → avanço/resolução → histórico/relatório
```

Mostrar o que ajuda a decidir:

- valor atual e saldo-base;
- produção e consumo do período;
- modificadores separados por origem;
- saldo final previsto;
- meta e prazo;
- pressupostos e eventos não incluídos;
- atualização após ação, evento e load.

Quando modificadores atuam em lados diferentes, explicar o efeito combinado. Por exemplo, produção a `80%` e consumo a `120%` não equivalem a uma única penalidade de `20%`: numa base `100/100`, o saldo vira `80 − 120 = −40`. A UI deve tornar essa pressão auditável sem exigir que o jogador reconstrua a fórmula.

Dificuldade precisa mostrar efeitos concretos vindos da regra ativa: reservas, metas, tolerância, requisitos, frequência e penalidades. Guia, criação de campanha, previsão e Teste Interno devem concordar. Não afirmar que um modo reduz custos ou aumenta produção quando esses multiplicadores são neutros.

Ao mudar balanceamento, comunicar o alcance temporal: regras derivadas podem valer em saves existentes; reservas iniciais ou geração de fundadores podem exigir campanha nova. Não sugerir retroatividade que o sistema não implementa.

## 11. Oportunidades e conteúdo procedural

Representar separadamente:

- checkpoint futuro ou alcançado;
- elegibilidade e requisito atual/necessário;
- pendência recuperável;
- oferta disponível;
- escolha aceita, recusada ou adiada;
- ativação/conclusão;
- oportunidade perdida somente quando a regra realmente a descarta.

Um resumo como `2/6` deve nomear o estado das quatro restantes. Mostrar se uma pendência antiga bloqueia as seguintes e quando haverá nova rechecagem. Recalcular após evento relevante e load; não deixar o painel depender apenas do próximo checkpoint para se atualizar.

Saves antigos precisam de estados seguros para campos novos e pendências acumuladas. Recuperar ofertas em sequência somente se essa for a regra aprovada; evitar abrir vários modais de uma vez ou apagar silenciosamente backlog.

Para cartas procedurais, testar várias seeds e exibir no diagnóstico:

- identidade e ID lógico;
- atributos, especialização e passiva;
- retrato/expressão correspondente;
- estado ativo/reserva/ofertado;
- motivo de bloqueio ou origem da oferta;
- seed e versão do gerador quando úteis à reprodução.

Capturar cartas expandidas é evidência de apresentação e de conteúdo, mas não prova distribuição global. Combinar amostra visual com contratos do gerador e campanhas reproduzíveis.

## 12. Antipadrões

- Ícone sem rótulo para ação desconhecida.
- Botão desativado sem motivo.
- Informação crítica apenas em tooltip.
- Modal para confirmação reversível comum.
- Toast como único registro de mudança importante.
- Toda ação com cor primária.
- Painel lateral que desaparece sem alternativa em tela estreita.
- HUD exibindo todos os sistemas sempre.
- Lista reconstruída a cada frame.
- Previsão sem pressupostos ou incerteza.
- Saldo extremo sem decomposição das causas.
- Guia ou dificuldade descrevendo regra diferente do runtime.
- Contador agregado que mistura futuro, bloqueado, pendente e concluído.
- Fallback plausível escondendo dado procedural inválido.
- Recomendação que muda dificuldade ou gasta recursos.
- Save que parece concluído antes da confirmação real.
- Escolha narrativa disparada no foco/hover.
