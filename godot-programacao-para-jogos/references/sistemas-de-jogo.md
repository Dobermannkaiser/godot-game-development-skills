# Sistemas de jogo orientados a dados

## Sumário

1. Regra geral
2. Economia e previsões
3. População, construções e filas
4. Dificuldade, metas e balanceamento
5. Narrativa e diálogos
6. Relacionamentos e recrutamento
7. Conteúdo procedural e aleatoriedade
8. Áudio
9. Desenho procedural e assets
10. Tutorial e guia
11. Checklist

## 1. Regra geral

Manter definição, estado, regra e apresentação separados:

```text
Definition/Resource → State → Rules/System → Event/Signal → View
```

Conteúdo deve possuir IDs estáveis e validadores. A UI nunca deve ser a única fonte de regra. Fórmulas usadas por gameplay, previsão, tooltip, IA e simulador devem vir da mesma implementação ou de uma API única para impedir divergência.

Uma fonte única de verdade não significa necessariamente uma função gigante. Pode haver calculadores por domínio, desde que runtime, previsão, relatório, teste interno e simulação chamem esses calculadores em vez de copiar constantes e fórmulas.

Antes de criar telemetria paralela, auditar o que o projeto já persiste: histórico diário, produção por personagem, decisões, obras e avaliações podem ser suficientes para montar relatórios auditáveis. Derivar apresentação desses fatos é mais seguro que manter um segundo contador que diverge do gameplay.

Ao aprofundar jogo existente, preservar escolhas aprovadas. Não adicionar automaticamente mecânica porque é comum no gênero. Explicar ao jogador qualquer redução, custo oculto, limite, cooldown ou regra de prioridade que afete decisão.

## 2. Economia e previsões

Modelar cada recurso com decomposição auditável:

```text
saldo = base + trabalhadores + construções + sinergias + eventos
        - consumo - manutenção - penalidades
```

Retornar resultado estruturado, não apenas total:

```gdscript
{
	"resource_id": &"food",
	"production": 42.0,
	"consumption": 31.0,
	"maintenance": 2.0,
	"event_delta": -4.0,
	"net": 5.0,
	"warnings": [&"low_reserve"]
}
```

Assim, previsão e relatório diário explicam o resultado. Manter arredondamento em fronteira definida; não arredondar em cada bônus e acumular erro sem intenção.

Para bônus multiplicativos/aditivos, definir ordem explicitamente e testá-la. Exemplo:

```text
produção final = (base + aditivos) × multiplicadores × retorno_decrescente
```

Não misturar porcentagem exibida e fator interno. Documentar caps, mínimo, stacking e condição de ativação.

Ao investigar um saldo extremo, decompor baseline e modificadores. Penalizar produção e aumentar consumo ao mesmo tempo produz pressão combinada: com ambos em `100`, fatores `0,80` e `1,20` geram `80 - 120 = -40`. Para reduzir essa pressão pela metade, `0,90` e `1,10` geram `-20`. Não dividir o saldo final mostrado, pois isso também perdoaria déficit-base, manutenção ou evento não relacionado.

Previsões devem distinguir:

- garantido pelo estado atual;
- estimado por tendência;
- condicionado a obra/evento;
- desconhecido por aleatoriedade.

Não apresentar estimativa como promessa.

Para metas futuras, incluir apenas decisões já comprometidas e consequências determinísticas que ocorrerão a tempo, como obra contratada com conclusão prevista. Não presumir futuras construções, profissões, escolhas narrativas ou eventos aleatórios. Exibir premissas e risco junto da projeção.

## 3. População, construções e filas

Separar população agregada de personagens nomeados. Definir se recrutamento altera população, capacidade, consumo e produção; não inferir.

Sistema de moradia pode envolver:

- capacidade;
- ocupação;
- atração;
- superlotação;
- manutenção;
- felicidade;
- pré-requisitos de expansão.

Para atração de população, registrar separadamente felicidade mínima, dias favoráveis consecutivos ou acumulados, capacidade, consumo incremental, progresso atual e regra de interrupção. A previsão precisa explicar por que a chegada avança, pausa ou reinicia. Ao rebalancear, testar save novo e save antigo, porque duração/limite derivados podem ter sido persistidos por versões anteriores.

Uma fila de construção precisa persistir:

- ID único da ordem;
- definição e nível/variante;
- custo já pago;
- momento enfileirado/iniciado;
- duração/progresso;
- prioridade;
- estado (`queued`, `active`, `completed`, `cancelled`);
- regras de reembolso;
- contexto necessário para migração.

Usar uma máquina de estados pequena e validar transições. Nunca deduzir apenas de `remaining_days == 0` se estados têm consequências diferentes.

Ao mudar capacidade simultânea:

- obras ativas normalmente continuam, se essa for a regra aprovada;
- impedir novas ativações acima do limite;
- tornar a prioridade visível;
- recalcular previsão sem reembolsar/cancelar silenciosamente.

Variantes irreversíveis devem ter:

- confirmação clara;
- comparação antes da escolha;
- ID persistente da variante;
- migração de saves anteriores;
- impossibilidade técnica de trocar sem ferramenta de debug, se a decisão é realmente permanente.

## 4. Dificuldade, metas e balanceamento

Centralizar dificuldade em dados e fórmulas. Não espalhar `if difficulty == ...` por UI e sistemas.

Alterar principalmente pressões pretendidas:

- metas;
- produção/consumo;
- custos/manutenção;
- margem de recuperação;
- severidade ou frequência de crises, quando isso fizer parte do design.

Não esconder informação ou tornar eventos injustos apenas para aumentar dificuldade, salvo intenção explícita. Regras qualitativas compartilhadas facilitam aprendizagem; números podem variar.

Para auditorias/metas:

```gdscript
const AUDIT_DAYS: Array[int] = [20, 40, 60]

var goals_by_difficulty: Dictionary = {
	&"moderate": {
		20: {&"population": 10, &"food": 45},
		40: {&"population": 15, &"food": 75},
	}
}
```

Validar que todos os dias/dificuldades possuem schema completo. A UI deve mostrar próxima meta, dias restantes, tendências e riscos explicáveis.

Balancear em três camadas:

1. análise de fórmulas e limites;
2. simulação com estratégias/seeds;
3. teste humano de clareza, ritmo e diversão.

Testar sem bônus, bônus médios e máximos. Um subsistema opcional como romance não deve ser obrigatório para vencer, salvo proposta explícita do jogo.

Testar especialmente o primeiro checkpoint, pois ele concentra pouca margem de correção. Verificar se:

- várias estratégias razoáveis conseguem chegar à meta;
- nenhuma construção única é obrigatória sem comunicação explícita;
- sementes iniciais não tornam a campanha quase condenada;
- composição procedural garante funções econômicas essenciais;
- dificuldade acolhedora oferece recuperação real, não apenas números ligeiramente menores.

Separar dificuldade de opacidade. Se alterar dias de atração, felicidade mínima, tolerância de crise ou metas, refletir os valores em guia, previsão e diagnóstico. Bônus oculto ou texto que descreve multiplicadores inexistentes destrói a leitura do sistema.

Medalhas/pontuação final podem considerar dificuldade, margens, crises e estado final, mas pesos devem ser transparentes o suficiente para evitar resultado arbitrário. Histórico global não deve ser apagado ao iniciar campanha.

Se o produto trocar pontuação/ranking por perfil descritivo ou medalhas comportamentais sem bônus, remover a semântica antiga de UI, histórico, save derivado, tutorial e verificadores ativos. Não manter dois modelos de encerramento concorrentes por compatibilidade acidental.

## 5. Narrativa e diálogos

Arquitetura recomendada:

- `DialogueDefinition`: conteúdo;
- `DialogueState`: progresso/flags;
- `DialogueRunner`: navegação e validação;
- `DialogueView`: apresentação;
- `StorySystem`: capítulos e consequências;
- sistemas de domínio: aplicam efeitos autorizados.

Estrutura mínima de nó:

```gdscript
{
	"id": &"intro",
	"speaker_id": &"npc_id",
	"text_key": &"dialogue.intro.line_1",
	"choices": [
		{
			"id": &"support",
			"text_key": &"dialogue.intro.support",
			"next": &"supported",
			"effects": [{"type": &"relationship", "amount": 10}]
		}
	]
}
```

Validar:

- ID inicial existente;
- nós alcançáveis;
- `next` válido ou término explícito;
- efeitos conhecidos e parâmetros válidos;
- condições sem referência ausente;
- ausência de loop não intencional;
- localização existente;
- retrato/voz quando obrigatórios.
- restrições editoriais e temas proibidos em todos os ramos, pools sazonais e falas procedurais, não apenas no caminho mais comum.

Não executar método arbitrário vindo dos dados. Usar registry/whitelist de efeitos com schema.

Para embaralhar respostas, embaralhar objetos completos; qualidade e consequência não podem depender da posição visual. Para conteúdo recorrente, guardar histórico curto e evitar repetição imediata quando houver alternativa, sem impedir repetição para sempre.

Definir ordem de eventos especiais. Exemplo:

```text
fechar economia diária → iniciar capítulo obrigatório → receber escolha
→ aplicar consequências → recrutamento → auditoria → vitória/derrota
```

A ordem é regra de produto e deve ter teste.

## 6. Relacionamentos e recrutamento

Manter pontos internos e marcos derivados. Não usar apenas “nível” se escolhas dependem de valores finos.

```gdscript
const MAX_RELATIONSHIP: int = 1000

func relationship_stage(points: int) -> int:
	return clampi(points / 200, 0, 5)
```

Definir:

- limites e deltas;
- ganho diário e cooldown;
- eventos únicos;
- flags de amizade/romance;
- exclusividade;
- condições de recrutamento;
- efeitos de saída/abandono;
- bônus de gestão e caps.

Resposta boa deve refletir personalidade e contexto, não ser sempre a opção mais gentil. Manter consequências legíveis e evitar penalização por informação que o jogador não podia conhecer, salvo intenção dramática clara.

Para romance:

- personagens adultos;
- consentimento e limites narrativos explícitos;
- regra de exclusividade consistente;
- amizade não apagada por rota rejeitada;
- campanha vencível sem romance, se romance for opcional;
- estado intermediário persistido para não duplicar evento.

Para recrutamento procedural ou condicionado:

- gerar candidatos antes da tela de escolha e persistir a oferta;
- salvar seed/IDs/atributos;
- não rerrolar ao recarregar;
- validar composição mínima se o jogo exige funções essenciais;
- mostrar por que candidatos apareceram e quando a oferta expira.

Separar seis conceitos que costumam ser confundidos:

1. **checkpoint:** o calendário cria o direito ou a tentativa;
2. **elegibilidade:** condições definem candidatos/fontes possíveis;
3. **oferta:** as opções são materializadas e persistidas;
4. **escolha:** o jogador aceita, recusa ou adia conforme a regra;
5. **ativação:** carta/personagem entra na equipe, reserva ou mundo;
6. **resolução:** o checkpoint deixa de estar pendente.

Não contar apenas personagens ativados quando a métrica de produto é “escolhas apresentadas”. Recusar troca de uma carta boa ou manter o candidato na reserva não deve apagar o fato de que a oferta ocorreu.

Todo requisito condicionado precisa de gatilhos de rechecagem. Se relacionamento pode destravar recrutamento, reavaliar após ganho real de relacionamento, após load e nos checkpoints relevantes. Reavaliar somente na próxima avaliação transforma uma diferença pequena em bloqueio de dezenas de dias.

Por padrão, um checkpoint antigo bloqueado não deve ocupar a frente de uma fila e impedir todos os posteriores. Se serialização estrita for parte do design, torná-la visível e testá-la. Caso contrário, manter pendências independentes ou aplicar catch-up em sequência segura. Saves existentes devem recuperar ofertas atrasadas sem rerrolar candidatos, duplicar recompensas ou conceder várias janelas simultâneas que a UI não suporta.

## 7. Conteúdo procedural e aleatoriedade

Procedural não significa irrestrito. Definir gerador com:

- seed;
- constraints;
- tentativas máximas;
- fallback determinístico;
- verificação pós-geração;
- versão do algoritmo quando resultado persistente precisa sobreviver a atualizações.

Exemplo de fundadores:

- atributos somam total fixo;
- mínimo/máximo por atributo;
- especializações variadas;
- passivas sem repetição;
- composição capaz de produzir recursos essenciais.

Se salvar apenas seed, mudar o algoritmo muda o mundo antigo. Para compatibilidade, salvar resultado materializado ou versão do gerador. Conteúdo cosmético e gameplay devem usar RNGs separados.

Uma identidade reproduzível de campanha deve registrar pelo menos semente e versão do gerador, e materializar resultados cujo algoritmo possa mudar. Aleatoriedade de acontecimentos que afeta gameplay deve derivar da identidade da campanha; animação, som e decoração podem usar streams separados.

## 8. Áudio

Um `AudioManager` global pode coordenar música, ambiente, SFX e UI, mas não deve conhecer regra de economia ou narrativa além de eventos semânticos.

Buses típicos:

```text
Master
├── Music
├── Ambience
├── SFX
└── UI
```

Práticas:

- mapear volume linear da UI para dB com tratamento correto de zero/mute;
- aplicar fade/crossfade sem criar players indefinidamente;
- evitar repetição imediata de faixas;
- restaurar música de contexto após faixa especial;
- fazer ducking durante diálogo importante;
- limitar sobreposição de sons frequentes;
- variar pitch discretamente apenas quando apropriado;
- cortar silêncio inicial do asset em vez de atrasar artificialmente a ação;
- documentar origem/licença;
- testar loop e volume percebido no motor.

OGG costuma ser adequado a música/ambiente e WAV a efeitos curtos, mas conferir plataforma, qualidade e import settings. Não afirmar validação auditiva com base apenas no arquivo existir.

## 9. Desenho procedural e assets

Desenho procedural é útil para formas escaláveis e paramétricas: grades, barras, molduras, muralhas, áreas e marcadores. Usar `_draw()` e `queue_redraw()`; calcular tudo de uma geometria central.

```gdscript
func _draw() -> void:
	_paint_wall_base()
	_paint_towers()
	_paint_gate()
```

Evitar helpers que colidam com métodos `draw_*` nativos. Separar dados geométricos de renderização para poder testar limites.

Não substituir arte aprovada por procedural apenas porque é tecnicamente simples. Escolher pela estética, escala, manutenção e performance. Para pixel art, respeitar filtragem, snapping, escala inteira e import settings do projeto.

## 10. Tutorial e guia

Estratégia em camadas:

1. tutorial inicial curto;
2. dicas contextuais na primeira necessidade;
3. guia completo consultável.

Persistir tutorial visto em configuração global quando deve valer para todas as campanhas. Permitir reabrir dicas/guia. Não bloquear controle sem rota de saída.

Atualizar conteúdo de ajuda na mesma entrega que muda mecânica relevante. O tutorial deve usar regras reais e não duplicar números que podem divergir; quando possível, ler valores de definições.

## 11. Checklist

- [ ] Previsão usa as mesmas fórmulas do runtime.
- [ ] Relatórios reutilizam fatos persistidos em vez de telemetria paralela.
- [ ] Saldo extremo é explicado por componentes e modificadores combinados.
- [ ] Projeção futura inclui somente compromissos determinísticos e explicita premissas.
- [ ] Custos, bônus e retornos são explicáveis.
- [ ] Fila de construção possui estados e IDs persistentes.
- [ ] Dificuldade está centralizada.
- [ ] Metas possuem schema completo e testes.
- [ ] Modelo final de pontuação ou perfil é único em UI, histórico e testes.
- [ ] Primeiro checkpoint é viável por mais de uma estratégia e por sementes válidas.
- [ ] Diálogos não executam callbacks arbitrários de dados.
- [ ] Escolhas não dependem da posição visual.
- [ ] Relacionamentos e recrutamentos não rerrolam após load.
- [ ] Checkpoint, elegibilidade, oferta, escolha, ativação e resolução não são confundidos.
- [ ] Dependências são reavaliadas no evento correto e pendências possuem catch-up.
- [ ] Conteúdo procedural possui constraints e fallback.
- [ ] Campanha registra semente, versão do gerador e streams de RNG coerentes.
- [ ] Áudio usa buses e não vaza players.
- [ ] Tutorial e guia refletem a mecânica atual.
