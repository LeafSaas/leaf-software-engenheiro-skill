# Operacao, Versionamento e Documentacao

## Fontes canonicas

Leia no repo atual antes de concluir tarefas:

- `INSTRUCT.md`: regra operacional de documentacao, versionamento e gates.
- `roadmap.md`: estado real do produto, notas de versao e pendencias.
- `test.map.md`: mapa vivo de rotas, modos de teste, prerequisitos e cenarios testaveis.
- `sprint-map*.md`, `*roadmap*.md`, `improvements.md`: planos de branch ou backlog quando existirem.
- `docs/gcp-release-flow.md` e `agente-golive.md`: deploy, GCP, Cloud Run, Cloud SQL, Scheduler, PIX e dominio.

Se houver divergencia, privilegie o arquivo mais especifico da branch/tarefa e registre a reconciliacao nos docs.

## Versionamento

Historico/pre-MVP usava:

- `demoalpha`
- `demobeta`
- `demorc`
- sufixos incrementais: `demobeta.1`, `demobeta.2`, etc.

Depois da saida de demo, usar SemVer:

- `vMAJOR.MINOR.PATCH`
- hotfix: `vMAJOR.MINOR.PATCH-hotfix` quando o repo/docs ja estiverem usando esse padrao.

Commit de release:

```bash
git commit -m "release: <versao>"
```

Toda versao nomeada precisa deixar uma nota curta no `roadmap.md` explicando o estado real da entrega.

## Roadmap e mapa de testes

Atualize `roadmap.md` quando houver:

- comportamento novo ou alterado;
- bugfix com impacto de produto;
- mudanca de status de feature;
- release, hotfix, deploy ou validacao operacional;
- lacuna conhecida descoberta durante a tarefa.

Atualize `test.map.md` quando houver:

- rota nova ou alterada;
- URL publica/admin mudando;
- modo de teste novo;
- prerequisito de ambiente;
- fluxo que passou a ser testavel;
- regressao, gap ou validacao manual pendente.

Use checklist visivel quando o usuario pedir acompanhamento incremental. Marque `[x]` somente quando o item estiver implementado e validado no nivel combinado.

## Branches e checkpoints

Antes de comecar trabalho grande:

- Checar `git status -sb`.
- Identificar branch atual e upstream.
- Preservar mudancas do usuario.
- Se o usuario pedir branch "up to date com a main", fechar docs pendentes na `main`, atualizar contra `origin/main` e so entao criar a branch.

Em branch com mapa aceito:

- Executar o plano da branch sem reabrir discussao de escopo.
- Atualizar o sprint map/roadmap da branch conforme itens fecham.
- Antes de merge/main, reconciliar `roadmap.md`, `test.map.md` e docs de release.

## Done definition

Uma entrega de codigo em `leaf-delivery` so esta pronta quando:

- comportamento implementado no escopo pedido;
- docs vivos atualizados;
- `npm run typecheck` e `npm test` executados, ou falha/bloqueio reportado com causa;
- testes especificos adicionados/ajustados quando houver bugfix, refactor estrutural ou comportamento novo;
- rotas ou runtime relevantes smocados quando o usuario pedir para rodar;
- migrations Prisma validadas quando schema ou producao forem afetados.

## Commit, push e deploy

So commitar/pushar quando o usuario pedir ou quando o fluxo aceito exigir.

Antes de commit:

```bash
git status -sb
npm run typecheck
npm test
```

Adicionar conforme risco:

```bash
npm run lint
npm run build
npx prisma validate
npm run db:prod:migrate:status
```

Para release GCP em `main`, o fluxo esperado e:

1. Validar localmente.
2. Verificar migrations de producao.
3. Aplicar migrations pendentes quando necessario.
4. Push para `main`.
5. Acompanhar Cloud Build/Cloud Run.
6. Smoke nas URLs publicas e admin.
7. Registrar status real nos docs.
