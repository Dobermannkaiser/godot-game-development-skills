# Acessibilidade e localização

## Sumário

1. Princípios e limites
2. Acessibilidade visual
3. Entrada e acessibilidade motora
4. Acessibilidade cognitiva
5. Movimento, flashes e tempo
6. Áudio, legendas e TTS
7. Leitores de tela no Godot
8. Localização, pluralização e RTL
9. Testes
10. Fontes oficiais

## 1. Princípios e limites

Tratar acessibilidade como remoção de barreiras a tarefas, não como lista cosmética. Priorizar pelo impacto:

1. bloqueia acesso ou causa perda;
2. impede compreender/decidir;
3. exige esforço desproporcional;
4. reduz conforto;
5. preferência estética.

WCAG 2.2 oferece critérios mensuráveis úteis, mas é padrão Web e não certifica automaticamente um jogo Godot. Diretrizes Xbox, Apple e outras precisam ser adaptadas à plataforma, gênero e input reais.

Não afirmar conformidade com base apenas em análise estática. Testar com jogadores e tecnologias assistivas quando o requisito for relevante.

Oferecer opções no primeiro acesso e nas configurações, sem esconder acessibilidade em submenu difícil. Defaults seguros não substituem personalização.

## 2. Acessibilidade visual

Usar como referência:

- texto comum: aproximadamente 4,5:1;
- texto grande: aproximadamente 3:1;
- componentes e ícones essenciais: aproximadamente 3:1 em relação ao adjacente;
- foco: claramente distinguível do estado sem foco e do fundo.

Medir as cores finais compostas, considerando transparência e fundo real. Shaders, gradientes, HDR e pós-processamento podem alterar o resultado.

Não depender apenas de cor. Combinar:

- forma;
- ícone;
- texto;
- padrão;
- posição;
- sinal `+/-`;
- movimento breve não essencial.

Texto sobre imagem:

- superfície opaca/semitransparente suficiente;
- sombra ou contorno moderado;
- região visual calma;
- modo de alto contraste/reduzir transparência quando necessário.

Escala:

- oferecer intervalos adequados ao projeto, por exemplo 80%–150%;
- reorganizar layout, não só ampliar bitmap;
- manter scroll e foco visíveis;
- testar textos e números extremos;
- evitar perder acesso ao botão que aplica/reverte a escala.

Alvos:

- WCAG 2.2 usa 24 × 24 CSS px como referência mínima com exceções;
- 44 × 44 é uma referência comum para toque confortável;
- não converter diretamente CSS px/pt em pixels de viewport Godot;
- medir no dispositivo e considerar espaçamento, alcance e frequência.

Oferecer, quando relevante:

- alto contraste;
- redução de transparência;
- escala de UI/texto;
- opções para daltonismo baseadas em redundância, não apenas filtros;
- cursor maior/mais visível;
- ajuste de brilho com padrão de calibração.

## 3. Entrada e acessibilidade motora

Garantir paridade para as entradas alvo. Nenhuma ação essencial disponível somente por:

- hover;
- clique direito;
- duplo clique rápido;
- segurar prolongado;
- gesto multi-dedo;
- arrasto preciso;
- combinação simultânea difícil.

Remapeamento, quando aplicável:

- permitir ações relevantes;
- detectar e explicar conflitos;
- atualizar glifos, tutorial e ajuda;
- preservar uma saída segura do modo de captura;
- restaurar padrões;
- armazenar configuração globalmente;
- considerar controles por jogador.

Alternativas úteis:

- toggle em vez de manter pressionado;
- atraso e velocidade de repetição configuráveis;
- sensibilidade e zona morta;
- selecionar origem/destino em vez de arrastar;
- ações contextuais por foco;
- pausa e tempo estendido;
- confirmação em uma pressão.

Para drag-and-drop, fornecer método de ponteiro simples ou navegação por foco quando o arrasto não for essencial.

## 4. Acessibilidade cognitiva

Reduzir memória de trabalho:

- objetivo atual;
- próximo passo;
- estado da seleção;
- custo e consequência;
- motivo de bloqueio;
- histórico recente;
- comparação lado a lado;
- ajuda contextual.

Usar linguagem simples e específica. Dividir instruções longas. Não usar ícone ambíguo sem rótulo. Manter nomenclatura constante entre tutorial, HUD e menus.

Foco ou seleção não deve aplicar decisão crítica automaticamente. Prévia pode acontecer; confirmação deve ser explícita e cancelável.

Dificuldade e assistência podem ser separadas:

- economia;
- velocidade;
- tempo limite;
- penalidades;
- intensidade do tutorial;
- dano/mira quando aplicável.

Descrever efeitos, não julgar o jogador. Evitar rótulos depreciativos para modos acessíveis.

Permitir reduzir notificações repetidas, pular conteúdo já visto e consultar histórico.

## 5. Movimento, flashes e tempo

Movimento deve comunicar origem/destino, estado, hierarquia ou feedback. Evitar animação contínua decorativa concorrendo com leitura.

Referências práticas, ajustáveis:

- microfeedback: 80–150 ms;
- estado de botão: imediato–120 ms;
- painel: 150–250 ms;
- troca de tela: 200–350 ms.

Não bloquear input durante toda transição quando não for necessário.

`Reduzir movimento` pode:

- substituir deslocamento/zoom/bounce por mudança instantânea ou fade curto;
- reduzir paralaxe, tremor e partículas;
- remover animação repetitiva;
- preservar feedback estático e estado final.

Controlar flashes e padrões. Não criar eventos luminosos intensos/repetitivos sem revisão específica de segurança. Oferecer redução de flashes quando aplicável.

Para tempo limite:

- informar antes;
- permitir pausar/estender/desativar quando o design permitir;
- não colocar leitura essencial sob cronômetro desnecessário;
- preservar progresso em interrupções.

## 6. Áudio, legendas e TTS

Sons de UI podem comunicar foco, confirmação, erro, aba e recompensa, mas nunca ser o único canal essencial.

Regras:

- bus separado para UI;
- volume previsível e normalizado;
- sons frequentes discretos;
- limitar sobreposição/repetição;
- feedback visual equivalente;
- respeitar mute e perda de foco;
- opção de reduzir sons repetitivos quando necessário.

Legendas:

- incluir fala essencial e identificar locutor;
- permitir tamanho, fundo/contraste e, quando útil, cor por personagem redundante ao nome;
- sincronizar com áudio sem velocidade impossível;
- não cobrir ação crítica;
- distinguir som importante não verbal quando necessário;
- não queimar legenda na imagem se precisa ser personalizável/localizada.

Godot possui TTS em plataformas suportadas, separado de leitor de tela. Detectar suporte e habilitar configuração do projeto antes de depender dele. TTS próprio não substitui árvore semântica para navegação por leitor de tela.

## 7. Leitores de tela no Godot

Versões recentes de Godot 4.x oferecem integração nativa com leitores de tela em desktop por AccessKit. Confirmar a versão, plataforma e template de exportação reais.

Em Godot 4.6, por exemplo:

- o suporte pode ser automático, sempre ativo ou desativado;
- `Control` oferece propriedades como nome e descrição acessíveis;
- controles padrão expõem semântica, mas interfaces customizadas ainda exigem trabalho;
- `FOCUS_ACCESSIBILITY` permite foco apenas quando leitor de tela está ativo;
- `AccessibilityServer` expõe roles, estados, relações, live regions e ações avançadas.

Não chamar diretamente APIs avançadas se propriedades de `Control` e controles padrão resolvem. Para custom controls, definir:

- papel/role correto;
- nome acessível curto;
- descrição somente quando adiciona informação;
- valor/estado (marcado, selecionado, expandido, desativado, ocupado);
- valor dinâmico com unidade, sinal, período e estado (`menos 12 alimentos por dia`, não apenas `-12`);
- ordem e relações de rótulo/descrição;
- ação acessível equivalente ao input;
- modal, região e live update quando apropriado;
- idioma do conteúdo quando muda.

Evitar:

- anunciar decoração;
- duplicar nome e descrição;
- usar tooltip como único nome;
- atualizar live region a cada frame;
- anunciar repetidamente a mesma previsão ou cada etapa intermediária de um recálculo;
- mover foco inesperadamente;
- esconder estado desativado;
- representar tabela/árvore complexa como sequência sem estrutura.

Fluxo de teste:

1. forçar modo de acessibilidade quando ferramenta não for detectada;
2. navegar com leitor de tela real em plataforma suportada;
3. verificar ordem, nomes, papéis, estados e ações;
4. abrir/fechar modal e confirmar restauração;
5. testar listas, tabs, sliders, árvore, texto e atualizações;
6. garantir que a versão exportada inclui dependências necessárias;
7. testar sem leitor de tela para detectar regressões de foco.

Suporte nativo não torna a tela acessível automaticamente. Interface customizada, ordem ruim e rótulos ausentes continuam barreiras.

Para recursos, metas e oportunidades, anunciar mudanças significativas e estáveis: meta atingida, risco alterado, requisito satisfeito, oferta disponível ou falha. Distinguir `zero`, `indisponível`, `não se aplica`, `pendente` e `erro`; o caractere visual `—` precisa de nome acessível quando o contexto não o torna inequívoco.

## 8. Localização, pluralização e RTL

Planejar desde o início:

- usar chaves de tradução;
- traduzir frases inteiras;
- usar pluralização e placeholders nomeados quando suportados;
- evitar concatenação gramatical;
- separar texto de ícones/imagens;
- permitir fontes e fallback por escrita;
- respeitar formato local de número, moeda, porcentagem, data e hora;
- manter contexto para tradutores.

Pseudolocalização deve revelar:

- expansão;
- texto sem tradução;
- caracteres ausentes;
- clipping;
- botões rígidos;
- alinhamento;
- direção RTL;
- ordem de foco inconsistente.

RTL:

- testar direção do layout e texto;
- espelhar apenas ícones direcionais sem significado cultural fixo;
- revisar anterior/próximo, breadcrumbs e gráficos;
- conferir ordem de foco e listas;
- não inverter números, barras ou cronologias automaticamente sem regra correta.

Não localizar IDs internos, caminhos de save ou valores usados como contrato. Localizar apenas apresentação.

Manter cálculo e valor bruto independentes da localização. Formatar separador decimal, agrupamento, sinal, plural e unidade somente na fronteira de apresentação; nunca reler o texto localizado para decidir regra. Testar números negativos, casas decimais, valores muito grandes e combinações como `atual / meta` em todos os idiomas alvo.

## 9. Testes

Matriz mínima:

```text
contraste normal/alto
cor desativada
UI 100/125/150%
sem mouse
controle remapeado
reduzir movimento
som desligado
legendas ampliadas
leitor de tela suportado
pseudolocalização
RTL aplicável
```

Ferramentas de simulação de daltonismo ajudam a encontrar riscos, mas não substituem medição nem teste com pessoas.

Registrar barreira, tarefa afetada, gravidade, frequência, evidência, solução e resultado do reteste.

## 10. Fontes oficiais

Confirmar documentação correspondente à versão do projeto. Referências consultadas na revisão de agosto de 2026:

- Godot `Control`: https://docs.godotengine.org/en/stable/classes/class_control.html
- Godot `AccessibilityServer`: https://docs.godotengine.org/en/stable/classes/class_accessibilityserver.html
- Godot — criação de aplicações e leitores de tela: https://docs.godotengine.org/en/stable/tutorials/ui/creating_applications.html
- Godot — navegação por teclado/controle: https://docs.godotengine.org/en/stable/tutorials/ui/gui_navigation.html
- Godot — texto para fala: https://docs.godotengine.org/en/stable/tutorials/audio/text_to_speech.html
- WCAG 2.2 Quick Reference: https://www.w3.org/WAI/WCAG22/quickref/
- Xbox Accessibility Guidelines: https://learn.microsoft.com/en-us/gaming/accessibility/xbox-accessibility-guidelines/

Usar `stable` somente depois de detectar a versão instalada; quando uma API muda, abrir a documentação versionada correspondente.
