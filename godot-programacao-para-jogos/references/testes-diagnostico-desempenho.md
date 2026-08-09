# Testes, diagnóstico e desempenho

## Sumário

1. Evidência e níveis de validação
2. Estratégia de testes
3. Execução headless
4. Testes de save e campanha
5. Ferramentas internas seguras
6. Diagnóstico e logging
7. Simulações determinísticas
8. Desempenho orientado por medição
9. Threads e WorkerThreadPool
10. CI e exportação
11. Checklist

## 1. Evidência e níveis de validação

Relatar separadamente:

- inspeção estática;
- parser/importação;
- teste automatizado;
- execução headless;
- execução visual;
- teste auditivo;
- exportação em plataforma-alvo;
- validação do usuário.

Nunca converter “não encontrei referência ausente” em “o jogo funciona”. Um parser simples não detecta lifecycle, foco, colisões nativas, warnings como erro, importação, comportamento de cena nem áudio.

Manter uma matriz de evidência explícita:

| Camada | Prova | Não prova |
| --- | --- | --- |
| busca textual/verificador próprio | contrato conhecido presente | sintaxe ou execução real |
| parser auxiliar | estrutura que ele entende | semântica completa do GDScript |
| compilador/importador Godot | scripts/resources aceitos e warnings reais | fluxo completo de gameplay |
| headless | cena e regras executam sem janela | layout, foco, áudio ou sensação |
| teste manual | comportamento observado no cenário | ausência de regressão em todos os casos |

Não somar contagens de verificadores como se fossem cobertura equivalente. Relatar o papel de cada um.

Quando o Godot não estiver disponível, ainda fazer diff, referências, schemas, IDs, scripts auxiliares e ZIP; declarar o limite.

## 2. Estratégia de testes

Construir pirâmide proporcional:

1. **regras puras:** cálculos, limites, seleção e invariantes;
2. **contratos de dados:** catálogos, Resources, IDs e schemas;
3. **integração:** cena + manager + estado + sinais;
4. **smoke:** projeto abre, cena principal carrega, erros críticos ausentes;
5. **fluxo completo:** campanha, save/load, vitória/derrota;
6. **manual visual/auditivo:** layout, input, animação, som e plataforma.

Manter regras importantes fora de nós visuais facilita teste com `RefCounted` ou objetos pequenos. Plugins de teste podem ser usados se já fizerem parte do projeto; não instalar addon sem autorização.

Cada bug corrigido deve, quando razoável, ganhar reprodução automatizada ou diagnóstico que impediria regressão. Não criar teste que apenas repete a implementação.

Converter o relato concreto em caso de regressão. Se o crash ocorreu ao formatar `48.0`, o teste precisa atravessar a fronteira que recebe `float`, não apenas procurar a presença de `str()` no arquivo. Se a falha envolve fila pendente, testar o evento que a bloqueia, o ganho que a destrava e o reload no meio.

Warnings do compilador fazem parte da qualidade. Cobrir nomes que sombreiam identificadores globais, declarações locais confundíveis, parâmetros/variáveis não usados e warnings configurados como erro. Um verificador customizado pode antecipá-los, mas não substitui recompilar no Godot.

## 3. Execução headless

Descobrir o executável:

```bash
command -v godot || command -v godot4 || true
```

Registrar versão real:

```bash
godot --version
```

Importar/abrir o editor sem janela, ajustando flags à versão detectada:

```bash
godot --headless --path /caminho/do/projeto --editor --quit
```

Executar uma cena de teste:

```bash
godot --headless --path /caminho/do/projeto --scene res://tests/smoke_test.tscn
```

Para argumentos do teste, preferir separador `--` e ler `OS.get_cmdline_user_args()` quando disponível na versão-alvo.

Cuidados:

- usar binário editor para importação/exportação;
- capturar exit code e logs completos;
- aplicar timeout externo razoável;
- tratar warnings/erros conhecidos de ambiente de forma explícita;
- não confundir falta de GPU/áudio com bug de gameplay;
- não usar `--headless` como prova de UI, input real ou áudio;
- verificar se export templates existem antes de exportar.
- registrar separadamente warnings e erros emitidos durante reload/importação;
- limpar apenas caches da cópia de teste quando a tarefa autorizar e quando isso for necessário para forçar recompilação; nunca apagar cache da base estável por rotina.

## 4. Testes de save e campanha

Manter fixtures pequenas e representativas para cada versão de schema suportada. Testar:

- novo jogo;
- round trip salvar → carregar → salvar;
- migração passo a passo;
- save mínimo, completo e corrompido;
- versão futura;
- backup/recovery;
- save durante diálogo, auditoria, recrutamento e troca de dia;
- operação idempotente após reload;
- configurações separadas.

Teste de campanha completa deve cobrir:

- tutorial;
- dias comuns;
- eventos e capítulos;
- construções e filas;
- relacionamentos e recrutamentos;
- estações/crises;
- todas as auditorias;
- vitória, derrota, medalha e retorno ao menu;
- save/load em pontos críticos.

Adicionar casos de compatibilidade comportamental quando catálogos mudam sem novo schema:

- regra de dificuldade reaplicada após load;
- progresso parcial preservado dentro do novo limite;
- ofertas/checkpoints atrasados recuperados sem rerrolar;
- item antigo bloqueado não apaga nem paralisa indevidamente os seguintes;
- previsão de save antigo usa a mesma regra do runtime atual.

Não depender só de clique manual para chegar a conteúdo raro. Criar fixtures ou ferramentas internas que preparem o estado com segurança.

## 5. Ferramentas internas seguras

Uma ferramenta de teste não pode contaminar a campanha real.

Fluxo obrigatório:

1. marcar sessão de diagnóstico;
2. capturar snapshot profundo de todo estado afetado;
3. suspender autosave e telemetria persistente;
4. preparar requisitos temporários;
5. executar o teste;
6. restaurar em bloco `finally` equivalente/saída garantida;
7. reconstruir a UI;
8. confirmar que nenhum arquivo foi gravado.

Se restauração perfeita não for simples, executar em estado/sessão temporária isolada, nunca sobre campanha real.

Ferramentas úteis:

- seletor de capítulo/diálogo;
- editor temporário de afinidade;
- estados de auditoria seguro/apertado/falha;
- avanço de dias com semente fixa;
- verificador de catálogos;
- inspeção de fila de eventos;
- overlay de performance e estado;
- captura de relatório exportável.

Esconder ou remover do export final por feature tag/configuração quando apropriado; não confiar apenas em botão invisível.

## 6. Diagnóstico e logging

Logs devem responder:

- o que aconteceu;
- em qual versão/cena/slot;
- com quais IDs e estado mínimo;
- qual regra decidiu;
- qual erro/código retornou.

Evitar logar a cada frame. Usar níveis e categorias coerentes. Não registrar dados pessoais, credenciais nem conteúdo sensível.

Um diagnóstico legível pode verificar:

- versão do jogo e schema;
- versão do motor;
- cena principal/autoloads;
- recursos e preloads ausentes;
- IDs duplicados;
- sinais/callbacks quebrados;
- personagens sem definição/retrato;
- diálogos com nós inalcançáveis;
- eventos sem condição válida;
- metas incompletas;
- flags incompatíveis;
- buses/faixas ausentes;
- estado de save e último backup.

O verificador deve espelhar condições reais. Se um campo só é obrigatório quando `requires_villager == true`, validar exatamente isso. Falso positivo recorrente destrói confiança na ferramenta.

Versionar ou rotular verificadores por contrato. Um teste histórico que exige nome, schema ou recompensa de etapa substituída deve ser aposentado ou atualizado, não contado como defeito atual nem mascarado. Antes de interpretar falha, confirmar que o verificador se aplica à versão-alvo.

## 7. Simulações determinísticas

Separar simulador da apresentação. Ele deve usar a mesma regra de domínio sempre que possível, sem duplicar fórmulas.

Registrar por execução:

- versão e parâmetros;
- semente;
- estratégia;
- decisões;
- saldo por recurso;
- falha, dia e causa;
- metas/auditorias;
- métricas agregadas.

Estratégias úteis para jogo de gestão:

- equilibrada;
- foco em população;
- foco em produção;
- construção precoce/tardia;
- sem relacionamentos;
- bônus máximos;
- sorte favorável/desfavorável;
- escolhas deliberadamente ruins.

Executar primeiro poucas sementes para detectar erro. Centenas ou milhares de campanhas são operação pesada: obter autorização quando não estiver inequivocamente pedida.

Simulação encontra impossibilidades, exploits e extremos; não prova diversão, clareza ou ritmo emocional. Comparar distribuição, não apenas média. Examinar pior percentil, taxa de falha, gargalos e sensibilidade a parâmetros.

Antes de simular milhares, criar testes matemáticos pequenos e auditáveis:

- baseline sem modificadores;
- cada modificador isolado;
- combinação de modificadores;
- limites e arredondamento;
- igualdade entre runtime, previsão e relatório;
- caso real informado pelo usuário.

Para penalidades em lados opostos da economia, medir a pressão combinada. Com produção e consumo-base iguais a `100`, produção `0,80` e consumo `1,20` geram saldo `-40`; trocar para `0,90` e `1,10` gera `-20`. Esse teste prova a redução do componente sazonal, mas não promete que todo déficit observado será exatamente dividido por dois se houver déficit-base, eventos ou manutenção.

## 8. Desempenho orientado por medição

Ordem de atuação:

1. reproduzir queda em build e cenário representativos;
2. medir com Profiler/Debugger/monitores;
3. separar CPU, GPU, física, navegação, memória e I/O;
4. localizar hotspot;
5. aplicar a menor mudança;
6. medir novamente com mesma cena/seed;
7. conferir regressão visual e de gameplay.

Primeiras correções geralmente melhores:

- eliminar trabalho por frame que pode reagir a sinal;
- desativar `_process`/`_physics_process` quando o nó está ocioso;
- reduzir criação/liberação repetitiva;
- evitar percorrer árvore e catálogos inteiros repetidamente;
- cachear cálculo caro com invalidação correta;
- repartir trabalho ao longo de frames;
- reduzir draw calls/overdraw somente após medir;
- carregar assets no momento adequado.

Não usar pooling universalmente. Pools aumentam complexidade e podem manter estado antigo; usar quando medição demonstra churn relevante e definir reset completo.

Usar `Performance.get_monitor()` ou monitores customizados para métricas reproduzíveis, mas remover/desativar coleta cara no release quando necessário.

## 9. Threads e WorkerThreadPool

Godot não torna toda a API do motor thread-safe. Regra padrão:

- fazer em worker: cálculo puro, parsing próprio, geração de dados, compressão ou preparação sem tocar SceneTree;
- fazer no thread principal: criar/modificar nós, cenas, UI, recursos ativos e APIs não declaradas thread-safe.

Antes de usar `Thread` ou `WorkerThreadPool`:

- provar que a tarefa é pesada e paralelizável;
- minimizar dados compartilhados;
- usar cópias imutáveis ou mutex com escopo pequeno;
- definir cancelamento e encerramento;
- coletar o resultado/aguardar finalização;
- transferir resultado para o thread principal com mecanismo seguro;
- tratar troca de cena/saída durante trabalho.

Criar threads sob demanda pode ter custo alto, especialmente em algumas plataformas. Preferir pool do motor quando compatível com a versão e o caso.

Nunca desligar verificações de thread safety para “resolver” erro sem demonstrar segurança. Deadlocks e race conditions exigem reprodução e instrumentação; não adicionar locks de forma aleatória.

## 10. CI e exportação

Pipeline mínimo possível:

1. checkout limpo;
2. instalar/selecionar versão exata do editor;
3. importar projeto headless;
4. executar testes;
5. exportar preset explícito;
6. verificar exit codes e artefatos;
7. guardar logs úteis.

Quando entregar um pacote-fonte, validar também o artefato final: gerar, reextrair em diretório temporário, comparar a árvore permitida e repetir os verificadores atuais sobre essa cópia. Testar apenas a árvore de trabalho não detecta exclusão acidental, arquivo velho ou lixo incluído no ZIP.

Não baixar motor ou templates sem autorização quando isso for operação material. Não commitar chaves de assinatura, keystores, certificados ou tokens. Separar presets versionáveis de segredos injetados pelo ambiente.

Exportar para uma plataforma não valida outra. Testar pelo menos cada família-alvo relevante, especialmente Web/mobile quando APIs, threads, filesystem, áudio ou shaders diferem.

## 11. Checklist

- [ ] Cada alegação de qualidade corresponde a evidência real.
- [ ] Compilador/importador real foi executado ou consta explicitamente como pendente.
- [ ] Warnings reais foram coletados e classificados.
- [ ] Regras puras possuem testes proporcionais.
- [ ] Saves suportados possuem fixtures de migração.
- [ ] Casos derivados de relatos reais atravessam a fronteira que falhou.
- [ ] Headless usa versão e flags confirmadas.
- [ ] Ferramenta interna não toca campanha real.
- [ ] Diagnóstico reflete as condições do runtime.
- [ ] Simulações registram seed e distribuição.
- [ ] Testes matemáticos isolam baseline, modificadores e combinações.
- [ ] Otimização possui medição antes/depois.
- [ ] Threads não acessam SceneTree indevidamente.
- [ ] CI/exportação não expõe segredos.
- [ ] Pacote reextraído repete as verificações aplicáveis à versão-alvo.
