---
name: leaf-software-engenheiro
description: Diretrizes de engenharia para trabalhar em projetos Leaf, especialmente leaf-delivery, cobrindo versionamento, roadmap, mapa de testes, padroes de desenvolvimento, validacao, arquitetura modular, documentacao obrigatoria, deploy GCP e operacao real. Use quando Codex for planejar, implementar, auditar, testar, versionar, publicar, atualizar documentacao ou diagnosticar tarefas em /home/gbezzu/Vibe/leaf-delivery ou projetos Leaf relacionados.
---

# Leaf Software Engenheiro

## Regra central

Trate os documentos do repositorio como fonte de verdade atual. Esta skill define o modo de trabalho; antes de editar codigo, leia os arquivos relevantes no checkout atual e confirme se algo mudou.

Prioridade de fontes em `leaf-delivery`:

1. Arquivo de instrucao citado pelo usuario: `AGENTS.md`, `AGENT*.MD`, `INSTRUCT.md`, `agente-golive.md`, roadmap de branch, sprint map ou documento equivalente.
2. Estado real do repo: `git status -sb`, branch atual, diffs existentes, scripts em `package.json`, schema Prisma e rotas/componentes envolvidos.
3. Documentos vivos: `roadmap.md`, `test.map.md`, `DESIGN.MD`, `ADMIN.DESIGN.md`, `CLIENT.DESIGN.md`, `COLORS.MD`, `docs/gcp-release-flow.md`, `refactor-god-obj.md`, `improvements.md`.
4. Esta skill e suas referencias.

## Fluxo obrigatorio

1. Levantar contexto antes de agir.
   - Ler o arquivo de instrucao citado pelo usuario quando houver.
   - Checar `git status -sb` e nao reverter mudancas que nao foram feitas por voce.
   - Identificar quais docs precisam acompanhar a entrega.
   - Para Next.js, ler a documentacao local relevante em `node_modules/next/dist/docs/` antes de usar APIs/convenções que possam ter mudado.

2. Planejar somente quando a tarefa pedir planejamento, auditoria, branch nova ou escopo aberto.
   - Se o usuario disser "Implement the plan", "siga as instrucoes" ou equivalente, executar diretamente.
   - Para branch de feature nova, manter ou criar roadmap/sprint map da branch antes de codar quando solicitado.

3. Implementar de forma cirurgica.
   - Reutilizar padroes locais, `src/components/ui`, contratos tipados, repository/domain modules e helpers existentes.
   - Evitar reescrita ampla quando uma correcao incremental resolve.
   - Nao introduzir infra futura como Redis, RabbitMQ, SSE, Kubernetes ou filas novas sem evidencia e pedido explicito.

4. Atualizar documentacao como parte do "done".
   - Mudanca de comportamento: atualizar `roadmap.md`.
   - Rota, fluxo, URL, pre-requisito ou criterio testavel: atualizar `test.map.md`.
   - UI/admin: atualizar `reforma.md`, `DESIGN.MD` e/ou `ADMIN.DESIGN.md`.
   - UI/cliente: atualizar `reforma.md`, `DESIGN.MD` e/ou `CLIENT.DESIGN.md`.
   - Cor/token/brand: atualizar `COLORS.MD`.
   - Deploy/GCP/runtime: atualizar `docs/gcp-release-flow.md` ou runbook ativo.

5. Validar no nivel certo.
   - Gate minimo para entrega de codigo: `npm run typecheck` e `npm test`.
   - Quando aplicavel: `npm run lint`, `npm run build`, `npx prisma validate`, `npm run db:prod:migrate:status`.
   - Se a entrega afetar producao com migrations: aplicar/registrar `npm run db:prod:migrate:deploy` antes de publicar.
   - Nao usar Playwright neste projeto; quando precisar de validacao visual/manual, pedir que o interlocutor abra as rotas e testar o caminho real por HTTP, scripts ou runtime local.

## Decisoes rapidas

- "Coloque pra rodar": iniciar app/stack necessaria, bater rotas reais e reportar URL/status; nao basta raciocinar estaticamente.
- "Hotfix" + "push na main": corrigir o menor escopo possivel, validar localmente, atualizar docs necessarios, commitar e publicar.
- "Verifique o que falta": auditar estado real contra roadmap/test map/design docs antes de propor trabalho.
- "Nao reescreve inteira": manter patch incremental e preservar fluxo existente.
- "Sem Playwright": usar testes unitarios, typecheck, build, curl/smoke HTTP e validacao manual guiada.
- Logs de producao ou erro real: diagnosticar pelo corpo da resposta/log, nao por suposicao.

## Referencias

Leia apenas a referencia necessaria para a tarefa:

- [references/operacao-versionamento.md](references/operacao-versionamento.md): versionamento, roadmap, test map, branch/release, docs obrigatorios e checkpoint.
- [references/arquitetura-testes.md](references/arquitetura-testes.md): stack, modulos, padroes de codigo, comandos, bancos, Prisma, GCP e runtime.
- [references/produto-ui.md](references/produto-ui.md): norte do produto, operacao, admin, storefront, mesa/comanda, WhatsApp e design system.
