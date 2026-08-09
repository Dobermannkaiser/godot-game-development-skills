# Persistência, compatibilidade e versionamento

## Sumário

1. Três versões diferentes
2. Schema de save
3. Migrações
4. Escrita robusta e recuperação
5. Carregamento defensivo e segurança
6. Transações de gameplay
7. Configurações globais
8. Compatibilidade do Godot
9. Controle de versão e arquivos do motor
10. Política de versões do projeto
11. Checklist

## 1. Três versões diferentes

Não confundir:

- **versão do motor:** Godot usado para abrir/importar/exportar;
- **versão do projeto/jogo:** entrega legível ao usuário, como `3.6.0`;
- **versão do schema de save:** inteiro monotônico usado por migrações.

Registrar todas quando forem relevantes. Uma atualização de conteúdo pode aumentar a versão do jogo sem mudar o schema; adicionar campo compatível com default pode ou não exigir migração, conforme contrato; mudar semântica ou estrutura persistida normalmente exige.

## 2. Schema de save

Todo save deve possuir envelope validável:

```gdscript
const SAVE_VERSION: int = 9

func build_save_data() -> Dictionary:
	return {
		"save_version": SAVE_VERSION,
		"project_version": ProjectSettings.get_setting("application/config/version", ""),
		"saved_at_unix": int(Time.get_unix_time_from_system()),
		"campaign": campaign_state.to_save_data(),
		"story": story_state.to_save_data(),
		"relationships": relationship_state.to_save_data(),
	}
```

Definir para cada campo:

- tipo esperado;
- default;
- limites;
- obrigatoriedade;
- primeira versão em que existe;
- regra de migração;
- se é derivado e pode ser recalculado.

Validar o contrato nos dois sentidos: a versão/schema exportada por cada subsistema deve ser aceita pelo respectivo carregador. Um produtor que grava versão `2` enquanto o consumidor só aceita `1` é incompatibilidade mesmo quando o envelope global permanece igual.

Evitar persistir cópias de regras configuráveis como se fossem progresso. Por exemplo, salvar `dias_necessarios_para_atracao` junto do contador pode fazer uma campanha antiga ignorar um rebalanceamento de dificuldade. Preferir salvar o contador e reaplicar a regra atual após load; se campanhas antigas precisarem conservar a regra anterior, persistir uma versão semântica explícita e migrar de forma deliberada.

Não salvar referências vivas a nós, Callables, sinais, recursos mutáveis de cena ou objetos que dependam do runtime. Salvar IDs e dados simples; reconstruir referências após carregar.

JSON é portável e inspecionável, mas não preserva todos os tipos do Godot e números podem perder distinções. Converter explicitamente `Vector2`, `Color`, enums, `StringName` e coleções tipadas. Formato binário pode ser menor/mais rápido, mas exige contrato e ferramentas de diagnóstico igualmente rigorosos.

## 3. Migrações

Migração deve ser:

- sequencial;
- determinística;
- idempotente no resultado final;
- tolerante a campos ausentes;
- validada com fixtures reais;
- incapaz de reduzir `save_version`;
- preservadora de dados desconhecidos quando possível.

Padrão:

```gdscript
func migrate_save_data(source: Dictionary) -> Dictionary:
	var data: Dictionary = source.duplicate(true)
	var version: int = int(data.get("save_version", 1))

	while version < SAVE_VERSION:
		match version:
			1:
				data = _migrate_v1_to_v2(data)
			2:
				data = _migrate_v2_to_v3(data)
			_:
				push_error("Não existe migração para a versão %d" % version)
				return {}
		version += 1
		data["save_version"] = version

	return data
```

Cada função migra exatamente um passo. Não apagar migradores antigos enquanto saves dessas versões forem suportados.

Testar:

- save mínimo de cada versão suportada;
- save completo;
- campo ausente;
- valor no limite;
- valor inválido recuperável;
- versão futura, que deve ser recusada ou aberta apenas em modo explicitamente seguro;
- migrar duas vezes e comparar resultado.

Quando uma atualização muda apenas a regra derivada, ainda testar saves existentes. Uma migração de schema pode ser desnecessária, mas o carregamento precisa reconstruir caches, limites, previsões e configurações derivadas com a fonte correta, preservando progresso válido sem conceder efeito retroativo não aprovado.

Não abrir save de versão futura como se fosse antigo. Isso pode apagar conteúdo desconhecido ao salvar novamente.

## 4. Escrita robusta e recuperação

Evitar sobrescrever o único save válido antes de concluir a nova escrita. Fluxo recomendado:

1. construir snapshot consistente em memória;
2. serializar e validar;
3. escrever em arquivo temporário dentro de `user://`;
4. fechar e conferir erros/tamanho;
5. mover o save atual para backup ou manter backup rotativo;
6. renomear o temporário para o destino;
7. somente então confirmar sucesso na UI.

Usar `FileAccess.get_open_error()` e códigos de `Error`; não tratar falha de abertura como arquivo vazio. Conferir resultados de `DirAccess.rename_absolute()`/remoção e diferenças de plataforma.

Política simples de recuperação:

- `slot_1.save`: principal;
- `slot_1.save.bak`: último principal válido;
- `slot_1.save.tmp`: escrita incompleta, nunca carregada automaticamente como principal.

Checksum detecta corrupção acidental, não prova autenticidade. Se houver ameaça de adulteração, definir requisito de segurança separado; não inventar criptografia caseira.

## 5. Carregamento defensivo e segurança

Ao carregar:

1. limitar caminho ao diretório e extensão permitidos;
2. limitar tamanho razoável;
3. analisar formato e capturar erro;
4. validar envelope, tipos e versão;
5. migrar uma cópia;
6. validar invariantes pós-migração;
7. construir novo estado fora da campanha ativa;
8. trocar o estado somente após sucesso completo.

Defaults protegem compatibilidade, mas não devem esconder corrupção estrutural. Distinguir:

- campo opcional ausente → default;
- valor antigo válido → migração;
- tipo inválido recuperável → normalização registrada;
- estrutura essencial corrompida → recusar e oferecer backup.

Nunca montar caminho diretamente de nome fornecido pelo usuário sem normalização. Rejeitar `..`, separadores inesperados e caminhos absolutos.

Tratar Resources/scripts externos como código potencialmente executável. Para saves ou mods não confiáveis, preferir formatos de dados com parser e whitelist de schema.

## 6. Transações de gameplay

Impedir save entre metade da consequência e metade das flags. Para diálogo obrigatório:

```text
marcar sequência pendente → persistir ponto seguro
→ receber escolha → calcular consequência
→ aplicar consequência e flag de conclusão juntas
→ enfileirar próximo passo → salvar estado consistente
```

Evitar duplicação após crash/load usando IDs de operação ou flags concluídas antes de conceder recompensa repetível. Recompensa importante deve ser idempotente:

```gdscript
func grant_once(reward_id: StringName, amount: int) -> bool:
	if reward_id in granted_reward_ids:
		return false
	granted_reward_ids.append(reward_id)
	resources += amount
	return true
```

Decidir explicitamente o que acontece ao sair durante:

- diálogo;
- recrutamento;
- auditoria;
- construção sendo concluída;
- troca de dia;
- vitória/derrota;
- tutorial.

Para filas, ofertas e eventos agendados, persistir estados distintos: `scheduled`, `eligible`, `offered`, `resolved`, `expired` ou equivalentes. Ao carregar:

1. reconstruir as regras derivadas;
2. validar itens pendentes e sua ordem;
3. recuperar checkpoints já alcançados;
4. materializar no máximo o que a UI consegue apresentar com segurança;
5. não rerrolar oferta já criada;
6. não duplicar recompensa concluída.

Definir política de catch-up. Itens atrasados podem ser apresentados em sequência, coexistir ou expirar; não deixar que o mais antigo bloqueie todos os posteriores por acidente. A ausência do evento original após load não pode impedir rechecagem permanente.

## 7. Configurações globais

Volume, idioma, remapeamento de input, opções de acessibilidade, tutorial visto e preferências de janela normalmente pertencem a arquivo global separado da campanha.

Aplicar configurações em duas fases quando houver risco:

- preview reversível;
- confirmação e persistência.

Manter defaults compatíveis com plataforma. Se o arquivo estiver corrompido, recuperar defaults sem tocar nos saves de campanha.

Não guardar senha/token em texto puro em configurações comuns. Usar armazenamento seguro da plataforma quando o projeto realmente possuir credenciais.

## 8. Compatibilidade do Godot

Antes de abrir com outra versão:

- identificar `config/features` e documentação do projeto;
- verificar plugins/GDExtensions;
- criar backup/commit;
- ler guia oficial de migração entre as versões exatas;
- abrir cópia separada;
- esperar diffs de reserialização em cenas/resources;
- validar importação, scripts, shaders, física, navegação, UI e export presets;
- confirmar export templates e plataformas.

Preferir patch estável compatível quando o projeto já está em uma linha suportada, mas não atualizar silenciosamente. Pré-releases só devem ser usadas com propósito explícito e cópia isolada.

Uma migração de motor não está concluída porque o editor abriu. Comparar comportamento, performance, warnings, renderização e exports.

## 9. Controle de versão e arquivos do motor

Versionar normalmente:

- `project.godot`;
- `.gd`, `.tscn`, `.tres`, shaders e assets-fonte;
- `addons/` necessários;
- `export_presets.cfg` quando não contém segredo;
- documentação e testes.

Ignorar normalmente:

- `.godot/` e caches de importação;
- builds/exportações geradas;
- logs, temporários e saves locais;
- credenciais e chaves de assinatura.

Arquivos `.tscn` e `.tres` em texto favorecem diff, mas merges automáticos ainda podem corromper IDs, ordem ou referências. Após conflito, importar no Godot e validar a cena.

Não renomear/mover muitos assets fora do editor sem conferir UIDs e referências. Não editar `.godot/uid_cache.bin` ou arquivos importados para “corrigir” referência.

Antes de qualquer edição, verificar alterações locais. Preservar tudo que não pertence à tarefa. Não usar comandos destrutivos para obter uma árvore “limpa”.

## 10. Política de versões do projeto

Seguir a convenção existente. Na ausência de outra regra:

- `X.Y.0`: nova etapa ou recurso significativo;
- `X.Y.Z`: correção/polimento compatível da etapa;
- schema de save: inteiro independente;
- base estável: somente versão aprovada pelo usuário.

Uma candidata mais nova não substitui automaticamente a base. Registrar origem da cópia e versão-alvo em relatório ou metadata existente.

Ao preparar pacote:

- excluir `.godot/`, cache, logs e saves pessoais;
- preservar `.godot` apenas se o projeto possuir razão explícita e documentada, o que é incomum;
- testar integridade do ZIP;
- gerar SHA-256;
- não incluir executável do motor ou dependências não licenciadas.

## 11. Checklist

- [ ] Motor, jogo e schema possuem versões distintas.
- [ ] Save é construído de snapshot consistente.
- [ ] Escrita temporária/backup evita perda do único arquivo válido.
- [ ] Erros de I/O são verificados.
- [ ] Migrações sequenciais possuem fixtures.
- [ ] Save futuro não é regravado destrutivamente.
- [ ] Operações críticas são idempotentes.
- [ ] Versões exportadas por subsistemas são aceitas pelos carregadores correspondentes.
- [ ] Dados derivados são reconstruídos a partir da fonte de verdade escolhida.
- [ ] Pendências atrasadas possuem política de recuperação e não duplicam ofertas/recompensas.
- [ ] Configurações globais estão separadas da campanha.
- [ ] Atualização do motor foi feita em cópia e conforme guia oficial.
- [ ] Cache e segredos não entram no controle de versão/pacote.
- [ ] A versão anterior aprovada permanece recuperável.
