# Entrega, revisão e critérios de conclusão

## Sumário

1. Antes de editar
2. Durante a implementação
3. Revisão de código
4. Revisão de saves e conteúdo
5. Revisão de UI e áudio
6. Validação e empacotamento
7. Roteiro de teste do usuário
8. Relatório de entrega
9. Critérios de conclusão
10. Falhas recorrentes

## 1. Antes de editar

- [ ] Confirmar pedido: análise, diagnóstico, plano, implementação ou entrega.
- [ ] Identificar última base testada e aprovada.
- [ ] Definir versão-alvo sem sobrescrever a estável.
- [ ] Ler instruções do projeto e status do controle de versão.
- [ ] Ler `project.godot`, cena principal e autoloads.
- [ ] Detectar versão/linha do Godot e plataformas.
- [ ] Mapear estado, regra, UI, conteúdo, save e testes afetados.
- [ ] Preservar mudanças locais fora do escopo.
- [ ] Registrar comportamento que não pode mudar.
- [ ] Definir critérios observáveis de conclusão.
- [ ] Registrar evidência concreta de bugs: mensagem, stack, linha, estado, save e semente disponíveis.
- [ ] Verificar se operação pesada/destrutiva exige autorização.

## 2. Durante a implementação

- [ ] Trabalhar em fatias pequenas e coerentes.
- [ ] Manter tipos e APIs compatíveis com versão mínima.
- [ ] Separar definição, estado, regra, coordenação e apresentação.
- [ ] Usar IDs estáveis e validar referências.
- [ ] Evitar caminhos frágeis e conexões duplicadas.
- [ ] Proteger `await`, timers e troca de cena.
- [ ] Não mutar Resource compartilhado por acidente.
- [ ] Atualizar save schema/migração quando necessário.
- [ ] Manter regra de previsão igual ao runtime.
- [ ] Recalcular dados derivados e reavaliar pendências após load.
- [ ] Conectar condições destraváveis aos eventos que realmente as alteram.
- [ ] Atualizar diagnóstico/testes junto com o recurso.
- [ ] Atualizar tutorial/guia quando mecânica muda.
- [ ] Conferir diff após cada bloco relevante.

## 3. Revisão de código

- [ ] Nenhum helper colide com API nativa/virtual.
- [ ] Nenhum callback ou sinal aponta para alvo ausente.
- [ ] Nenhum preload/resource/path está quebrado.
- [ ] Retornos `Variant` são convertidos/validados.
- [ ] Valores numéricos não usam `String(variant)` como conversor universal.
- [ ] Warnings não foram silenciados amplamente.
- [ ] Identificadores não sombreiam funções globais nem repetem nomes confundíveis em blocos próximos.
- [ ] Parâmetros/variáveis não usados foram removidos, renomeados ou justificados no menor escopo.
- [ ] Estado não depende diretamente da UI.
- [ ] Autoload não virou controlador de todos os domínios.
- [ ] Eventos são emitidos após estado consistente.
- [ ] Operações críticas são idempotentes.
- [ ] Não existe polling por frame sem necessidade.
- [ ] Nenhuma thread toca SceneTree sem suporte explícito.
- [ ] Erros de I/O e códigos `Error` são tratados.
- [ ] Logs não vazam dados sensíveis.
- [ ] Código antigo aprovado não foi removido sem intenção.

## 4. Revisão de saves e conteúdo

- [ ] `save_version` e versão do projeto estão corretos.
- [ ] Defaults são centralizados e defensivos.
- [ ] Todas as versões suportadas têm cadeia de migração.
- [ ] Fixtures antigas continuam carregando.
- [ ] Versão futura é recusada com segurança.
- [ ] Escrita usa temporário/backup quando risco justifica.
- [ ] Save durante sequência obrigatória não duplica efeito.
- [ ] Configurações globais estão separadas.
- [ ] IDs de catálogos são únicos e estáveis.
- [ ] Diálogos não possuem nós/efeitos inválidos.
- [ ] Conteúdo procedural persiste resultado ou versão do gerador.
- [ ] Recrutamento/aleatoriedade não rerrola após load.
- [ ] Versão/schema exportado por cada subsistema é aceito pelo carregador.
- [ ] Regra derivada atual e progresso persistido não entram em conflito.
- [ ] Checkpoints/ofertas atrasados possuem recuperação sem bloquear os posteriores.

## 5. Revisão de UI e áudio

- [ ] Resolução mínima permanece usável.
- [ ] Texto longo/localizado não corta ação essencial.
- [ ] Modais bloqueiam input abaixo.
- [ ] Janela invisível não intercepta mouse.
- [ ] Foco inicial, cancelamento e retorno funcionam.
- [ ] Teclado/controle não dependem de mouse.
- [ ] Scroll ocupa somente área disponível.
- [ ] Listas mantêm seleção por ID.
- [ ] Estado assíncrono não atualiza tela fechada.
- [ ] Volume usa buses/configuração persistente.
- [ ] Faixas/players não se acumulam.
- [ ] Loop, atraso e volume percebido foram testados no motor ou marcados como pendentes.

## 6. Validação e empacotamento

Executar conforme ambiente e escopo:

- [ ] Inspecionar diff final e arquivos não relacionados.
- [ ] Procurar nomes/versões/IDs antigos quando houve renomeação.
- [ ] Verificar referências e recursos ausentes.
- [ ] Validar catálogos e schemas.
- [ ] Rodar parser/importação Godot.
- [ ] Coletar e classificar warnings do compilador real.
- [ ] Rodar testes puros e integração.
- [ ] Rodar fixtures de save/migração.
- [ ] Rodar smoke headless.
- [ ] Executar simulação curta com seed fixa, se relevante.
- [ ] Testar visual/input/áudio, se motor e interface estiverem disponíveis.
- [ ] Exportar apenas presets pedidos.
- [ ] Conferir exit codes e logs.

Não substituir importação/compilação real por contagem de contratos estáticos. Se o Godot não foi executado, registrar explicitamente que warnings e runtime permanecem para teste do usuário.

Ao empacotar projeto:

- [ ] Usar exatamente a versão-alvo correta.
- [ ] Excluir `.godot/`, cache, logs, temporários e saves pessoais.
- [ ] Excluir segredos, keystores e certificados.
- [ ] Incluir addons e assets licenciados necessários.
- [ ] Verificar listagem e integridade do ZIP.
- [ ] Reextrair em diretório temporário e comparar o conteúdo permitido.
- [ ] Repetir verificadores atuais sobre a árvore reextraída.
- [ ] Calcular SHA-256.
- [ ] Conferir nome do arquivo e versão.
- [ ] Manter base anterior disponível.

## 7. Roteiro de teste do usuário

Criar roteiro curto e específico, sem pedir que o usuário teste o jogo inteiro. Formato:

```text
Pré-condição:
- iniciar save novo / carregar fixture X;
- usar resolução Y;
- abrir tela Z.

Passos:
1. executar ação observável;
2. confirmar texto/estado;
3. salvar e recarregar;
4. repetir caso-limite.

Resultado esperado:
- comportamento A;
- nenhum erro nem warning novo no debugger;
- estado B preservado após load.
```

Priorizar pontos que o ambiente não validou: sensação, foco real, resolução, áudio, controle, importação local e continuidade do save.

Não transformar o usuário em testador genérico. Indicar exatamente onde olhar e quanto deve levar.

## 8. Relatório de entrega

Modelo:

```text
Concluída a versão X.Y.Z, criada a partir da base estável A.B.C.

Implementado
- mudança observável 1;
- mudança observável 2;
- compatibilidade/migração de save;
- diagnóstico ou teste adicionado.

Validação executada
- verificação: resultado;
- Godot headless: resultado/não disponível;
- teste visual/áudio: resultado/não executado;
- ZIP/checksum: resultado, quando aplicável.

Teste manual recomendado
1. passo curto;
2. resultado esperado.

Limitações e pendências
- somente itens reais e acionáveis.
```

Evitar dizer “sem bugs” ou “perfeito”. Dizer “nenhum erro encontrado nas verificações X” é verificável.

## 9. Critérios de conclusão

Um recurso está pronto para entrega quando:

1. atende ao comportamento acordado;
2. integra-se sem reformular sistemas aprovados;
3. mantém invariantes;
4. salva/carrega ou declara que não afeta persistência;
5. possui UI/input/acessibilidade compatíveis com o escopo;
6. possui conteúdo e assets necessários;
7. tem diagnóstico/teste proporcional ao risco;
8. passa verificações disponíveis;
9. tem limites não testados declarados;
10. pode ser testado rapidamente pelo usuário;
11. preserva a base anterior;
12. aguarda aprovação antes de virar base estável.

## 10. Falhas recorrentes

### Trabalhar na versão errada

Sintoma: recursos aprovados desaparecem. Prevenção: registrar hash/versão/origem antes de copiar.

### Corrigir sintoma com defer/validade

Sintoma: erro some mas UI fica atrasada ou inconsistente. Prevenção: mapear lifecycle e propriedade do estado antes de usar `call_deferred()` ou checks.

### Alterar schema sem migração

Sintoma: saves antigos abrem com estado parcial. Prevenção: fixtures e migradores sequenciais.

### Salvar no meio da consequência

Sintoma: recompensa duplica após load. Prevenção: transação lógica e IDs idempotentes.

### Modal só visualmente no topo

Sintoma: aparece, mas clique atravessa ou não funciona. Prevenção: camada, blocker, mouse filter e foco no contêiner inteiro.

### UI calculando regra própria

Sintoma: previsão diverge do dia resolvido. Prevenção: API única de cálculo detalhado.

### Resource compartilhado como estado

Sintoma: alterar personagem muda outro. Prevenção: separar definição/estado e duplicar/localizar Resource.

### Ferramenta de debug grava campanha

Sintoma: dias/recursos ficam alterados. Prevenção: sessão isolada, autosave suspenso e restauração garantida.

### Otimizar sem medir

Sintoma: código complexo sem ganho. Prevenção: cenário, profiler e comparação antes/depois.

### Validador simplista

Sintoma: falsos positivos ignorados pela equipe. Prevenção: refletir exatamente schema e condições do runtime.

### Pacote com cache/segredo

Sintoma: ZIP enorme ou vazamento. Prevenção: allowlist de conteúdo, inspeção e checksum.

### Verificador estático aprovado, compilador real falha

Sintoma: contagens de contratos passam, mas Godot mostra warning ou crash ao abrir a tela. Prevenção: separar evidências, compilar/importar no motor quando autorizado e transformar o valor real que falhou em regressão.

### Conversão de `Variant` derruba painel

Sintoma: `Invalid call 'String' constructor` com `float`. Prevenção: formatador tipado na fronteira da UI, `str()` como fallback e testes com inteiros/decimais/save antigo.

### Regra atual não alcança save antigo

Sintoma: catálogo foi rebalanceado, mas valor persistido mantém comportamento anterior. Prevenção: persistir progresso, reconstruir derivados após load e testar compatibilidade comportamental mesmo sem mudar schema.

### Pendência antiga bloqueia agenda inteira

Sintoma: checkpoints posteriores existem, mas nenhuma nova oferta aparece. Prevenção: separar checkpoint/elegibilidade/oferta/resolução, reavaliar no evento correto e definir catch-up sem bloqueio acidental.

### Penalidades combinadas excedem intenção

Sintoma: produção reduzida e consumo aumentado geram déficit muito maior que a leitura isolada de cada percentual. Prevenção: decompor baseline, testar combinação e alterar o modificador causal em uma fonte única de verdade.
