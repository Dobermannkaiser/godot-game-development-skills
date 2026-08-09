# Fundamentos, fluxos e conteúdo

## Sumário

1. Modelo de UX
2. Pesquisa e descoberta
3. Jornada e arquitetura de informação
4. Hierarquia e divulgação progressiva
5. Prevenção e recuperação de erros
6. UX writing e ajuda
7. Onboarding
8. Interfaces adaptativas
9. Artefatos de projeto
10. Contratos de dados, tempo e pendências

## 1. Modelo de UX

Tratar UX como a experiência completa de compreender, decidir, agir, receber resposta e recuperar-se de erros. UI é a camada visual e interativa dessa experiência. Não confundir beleza com usabilidade.

Antes de modificar uma tela, responder:

1. Qual tarefa o jogador tenta concluir?
2. Que informação precisa para decidir?
3. Qual erro tem maior custo?
4. O que precisa permanecer visível?
5. O que pode aparecer sob demanda?
6. Como cancelar ou voltar?
7. Como sucesso, falha, bloqueio e andamento serão comunicados?

Usar as heurísticas como lentes:

- estado do sistema visível;
- linguagem do jogador e do mundo do jogo;
- controle e liberdade;
- consistência;
- prevenção de erros;
- reconhecimento em vez de memorização;
- eficiência para iniciantes e experientes;
- minimalismo funcional;
- mensagens que ajudam a recuperar;
- ajuda disponível no contexto.

## 2. Pesquisa e descoberta

Antes de propor solução, reunir evidência proporcional:

- versão estável e objetivo do jogo;
- público, gênero, plataformas e duração das sessões;
- resolução base e mínima;
- entradas suportadas;
- telas, modais, HUD, tutorial, configurações e save;
- identidade visual, fontes, arte e limitações;
- tarefas frequentes e tarefas raras de alto risco;
- dúvidas e erros observados pelo usuário;
- telemetria, relatos, gravações ou capturas disponíveis;
- stacks, warnings, valores exatos, semente, dia, dificuldade e versão do save quando houver falha reproduzível;
- requisitos de acessibilidade e localização.

Não inventar pesquisa com jogadores. Quando ela não existir, nomear a proposta como hipótese de design e fornecer um teste para validá-la.

Inventário de tela:

```text
Tela/estado:
Tarefa:
Ação principal:
Ações secundárias:
Informação crítica:
Entradas:
Vazio/carregando/erro:
Saída:
Risco:
Evidência:
```

## 3. Jornada e arquitetura de informação

Descrever a tarefa como passos do jogador, não como nós ou scripts:

```text
quero construir
→ encontro construções
→ comparo opções
→ entendo custo e prazo
→ confirmo
→ recebo feedback
→ vejo a fila e a previsão atualizadas
```

Marcar:

- primeiro ponto de orientação;
- decisões e pré-requisitos;
- memória exigida entre telas;
- retornos e becos sem saída;
- pontos de hesitação;
- consequências irreversíveis;
- feedback atrasado;
- mudança de contexto.

Organizar conteúdo conforme o modelo mental do jogador. Nomes internos como `ResourceManager` ou `CycleResolution` não definem categorias de interface. Agrupar por tarefa, frequência e relação.

Manter informação perto da decisão:

- custo ao lado de comprar/construir;
- valor atual e necessário juntos;
- prazo e previsão perto da fila;
- efeito da escolha antes da confirmação;
- motivo no próprio bloqueio;
- comparação na mesma tela, não em memória.

Listas crescentes podem precisar de busca, ordenação, filtros, agrupamento e estado vazio. Não adicionar essas funções a listas pequenas sem benefício real.

## 4. Hierarquia e divulgação progressiva

Definir por contexto:

1. informação primária;
2. ação primária;
3. informações de apoio;
4. ações secundárias;
5. detalhe avançado;
6. ação destrutiva.

Usar tamanho, peso, espaçamento, posição, superfície, cor e movimento para criar hierarquia. Não usar saturação máxima em tudo.

Divulgação progressiva:

- primeiro nível: identidade, estado, efeito principal e ação;
- segundo nível: comparação, requisitos, produção, prazo e manutenção;
- ajuda: regra completa, exceções e exemplos.

Não esconder informação crítica apenas em hover, tooltip, gesto, ícone ou tela distante. Em toque e controle, hover pode não existir.

Minimalismo funcional remove ruído, não contexto. Preservar rótulos, consequência, estado, contraste e caminho de retorno.

## 5. Prevenção e recuperação de erros

Priorizar pelo custo:

- perder campanha ou save;
- gastar recurso raro;
- encerrar turno/dia crítico;
- escolher resultado narrativo irreversível;
- alterar dificuldade ou configuração de vídeo arriscada;
- descartar trabalho não salvo.

Prevenir com:

- limites e validação antes da ação;
- prévia de custo e efeito;
- padrão seguro;
- botão temporariamente bloqueado contra clique duplo;
- ação destrutiva afastada da principal;
- alternativa ao arrasto;
- confirmação apenas quando o custo justifica interrupção;
- desfazer ou período de reversão quando viável.

Uma confirmação deve nomear a ação e a perda:

```text
Sobrescrever o Slot 2?
O progresso existente será substituído.

[Sobrescrever] [Cancelar]
```

Mensagem de erro útil:

1. o que aconteceu;
2. causa conhecida ou condição observada;
3. como tentar resolver;
4. o que foi preservado.

Evitar códigos internos sem tradução. Um código pode aparecer como detalhe copiável para suporte.

## 6. UX writing e ajuda

Usar linguagem direta, consistente e traduzível.

- Botões: verbos específicos como `Construir`, `Salvar`, `Reordenar`, `Cancelar`.
- Bloqueio: `Requer 30 materiais. Você possui 18.`
- Andamento: `Salvando…` seguido por confirmação ou erro.
- Risco: dizer consequência e prazo.
- Números: mostrar unidade, sinal, período, precisão e comparação quando relevantes; formatar inteiros e decimais de modo deliberado.

Evitar:

- `OK` para ações diferentes;
- `Sim/Não` sem repetir a decisão;
- termos técnicos internos;
- humor que esconda solução em erro crítico;
- textos longos em modais;
- frases montadas por concatenação.

Respeitar limites de conteúdo definidos pelo projeto também em geração procedural, variantes, fallback e localização. Uma frase proibida não pode reaparecer por outro catálogo, personalidade ou caminho de teste. Auditar o corpus completo quando o tema for sensível; não confiar apenas na tela observada.

Tooltips:

- disponibilizar por foco e ponteiro;
- permitir dispensar;
- permanecer enquanto houver intenção;
- não cobrir o alvo ou informação essencial;
- não ser a única fonte de informação crítica;
- incluir atalho quando útil.

Estruturar ajuda:

1. onboarding curto do ciclo essencial;
2. dica contextual na primeira necessidade;
3. guia pesquisável ou bem categorizado;
4. histórico ou glossário quando o jogo é sistêmico.

## 7. Onboarding

Ensinar no momento de uso:

1. apresentar objetivo imediato;
2. destacar uma ação possível;
3. permitir que o jogador execute;
4. confirmar resultado;
5. liberar o fluxo;
6. manter a explicação no guia.

Evitar páginas de instrução antes de o jogador ter contexto. Permitir pular, rever e reduzir dicas repetidas. Não mascarar uma interface confusa com tutorial permanente.

Para mecânicas novas, atualizar juntos:

- rótulos e ajuda;
- estado vazio;
- primeira aparição;
- mensagens de bloqueio;
- feedback de sucesso;
- guia e histórico.

## 8. Interfaces adaptativas

Uma adaptação segura é:

- relevante para o estado atual;
- previsível e estável;
- explicável;
- reversível ou ignorável;
- persistida somente quando é preferência;
- limitada por frequência;
- respeitosa da privacidade.

Pode:

- destacar meta em risco;
- ordenar alertas por urgência;
- lembrar última aba ou filtro;
- adaptar glifos ao dispositivo ativo;
- reduzir movimento;
- sugerir uma ação e explicar o motivo.

Não pode, sem ação explícita:

- confirmar decisão crítica;
- gastar recursos;
- reorganizar controles principais de modo imprevisível;
- alterar dificuldade;
- escolher resposta narrativa;
- ocultar informação para manipular comportamento.

Registrar anti-repetição de sugestões: última exibição, cooldown, condição resolvida, dispensas e preferência de ajuda.

## 9. Artefatos de projeto

Produzir somente o necessário:

- mapa de jornada;
- inventário de telas;
- wireframe anotado;
- contrato de componente;
- tabela de estados;
- fluxo de foco;
- matriz de resolução e entrada;
- relatório de auditoria;
- roteiro de teste.

Wireframe deve provar hierarquia, tarefa, navegação, scroll, conteúdo extremo e resolução mínima. Não precisa de arte final.

Especificação de componente:

```text
Nome e objetivo
Conteúdo e limites
Estados
Entradas e ações
Foco inicial e vizinhos
Comportamento responsivo
Semântica acessível
Sinais emitidos
Som/movimento
```

## 10. Contratos de dados, tempo e pendências

Definir a semântica antes de escolher o texto visual:

- `0`: valor conhecido e nulo;
- vazio: coleção válida sem itens;
- ausente: campo ou entidade não fornecido;
- não aplicável: regra deliberadamente não se aplica;
- bloqueado: existe, mas falta condição;
- pendente/adiado: checkpoint ocorreu e aguarda rechecagem ou decisão;
- erro: dado inválido ou operação falhou.

Não transformar ausência ou erro em um rótulo plausível como `Sem profissão`, `Nenhum bônus` ou `0`. O fallback precisa preservar a verdade para o jogador e deixar o defeito detectável no diagnóstico.

Para sistemas temporais, modelar e nomear etapas separadas:

```text
checkpoint → elegibilidade → pendência → oferta → escolha → ativação → resolução
```

Uma etapa não pode assumir que as seguintes ocorreram. Definir para cada uma: gatilho, condição, estado persistido, evento de rechecagem, recuperação após load e apresentação. Uma pendência antiga não deve bloquear silenciosamente marcos posteriores sem que isso seja regra explícita e comunicada.

Contadores precisam explicar numerador e denominador. `2/6 concluídos` não informa sozinho se quatro oportunidades estão futuras, bloqueadas, adiadas, recusadas ou perdidas. Mostrar a composição quando ela muda a decisão e indicar o próximo gatilho relevante, sem prometer um evento que a regra ainda não garante.
