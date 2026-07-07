---
name: leaf-software-engenheiro
description: Diretrizes de engenharia para trabalhar em projetos Leaf, especialmente leaf-delivery, cobrindo versionamento, roadmap, mapa de testes, padroes de desenvolvimento, validacao, seguranca (auth/tenant/IDOR/webhooks/segredos/erros), arquitetura modular, documentacao obrigatoria, deploy GCP e operacao real. Use quando Codex for planejar, implementar, auditar, testar, versionar, publicar, revisar seguranca, atualizar documentacao ou diagnosticar tarefas em /home/gbezzu/Vibe/leaf-delivery ou projetos Leaf relacionados.
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
   - **Seguranca em entrega que toca auth, tenant/empresa, banco, pagamento, webhook, cron ou endpoint publico:** aplicar `references/seguranca.md`. As duas falhas mais comuns de codigo gerado por LLM: (1) **IDOR** (rota com `:id` que carrega o recurso sem filtrar pelo dono, ex.: `findUnique({ where: { id } })` em vez de `findFirst({ where: { id, userId } })`); (2) **autorizacao no claim** (decidir `role`/`plano`/`isAdmin` a partir do token/body do cliente em vez de reler do banco). Toda rota nova com `:id` nasce com filtro por dono e teste cross-tenant. Webhook verifica assinatura; segredo de cron/job/api-key compara `timingSafeEqual` sem fallback; erro do Prisma nunca vai cru ao cliente.

4. Atualizar documentacao como parte do "done".
   - Mudanca de comportamento: atualizar `roadmap.md`.
   - Rota, fluxo, URL, pre-requisito ou criterio testavel: atualizar `test.map.md`.
   - UI/admin: atualizar `reforma.md`, `DESIGN.MD` e/ou `ADMIN.DESIGN.md`.
   - UI/cliente: atualizar `reforma.md`, `DESIGN.MD` e/ou `CLIENT.DESIGN.md`.
   - Cor/token/brand: atualizar `COLORS.MD`.
   - Deploy/GCP/runtime: atualizar `docs/gcp-release-flow.md` ou runbook ativo.
   - **README.md raiz e a fonte de verdade mais consultada do sistema.** Quando a entrega mudar
     versao/live, rota, integracao/homologacao, stack, env, script ou estrutura de pastas, atualizar
     o `README.md` raiz no mesmo conjunto de mudancas, seguindo o metodo de
     `references/readme-fonte-de-verdade.md` (atualizar, nunca reescrever do zero; manter e expandir
     os Mermaids; badges por fileira; status honesto de homologado x em breve x planejado).

5. Validar no nivel certo.
   - Gate minimo para entrega de codigo: `npm run typecheck` e `npm test`.
   - Quando aplicavel: `npm run lint`, `npm run build`, `npx prisma validate`, `npm run db:prod:migrate:status`.
   - Se a entrega afetar producao com migrations: aplicar/registrar `npm run db:prod:migrate:deploy` antes de publicar.
   - Validacao visual/manual: Playwright esta disponivel para abrir a rota, capturar screenshot e inspecionar regressao visual (ver o workflow visual em `AGENTS.md`). Sem virar suite de specs. Complementar sempre com testes, typecheck, build, curl/smoke HTTP e runtime local.

## Decisoes rapidas

- "Coloque pra rodar": iniciar app/stack necessaria, bater rotas reais e reportar URL/status; nao basta raciocinar estaticamente.
- "Hotfix" + "push na main": corrigir o menor escopo possivel, validar localmente, atualizar docs necessarios, commitar e publicar.
- "Verifique o que falta": auditar estado real contra roadmap/test map/design docs antes de propor trabalho.
- "Nao reescreve inteira": manter patch incremental e preservar fluxo existente.
- Validacao visual: Playwright pode ser usado para screenshot/regressao visual (sem virar suite de specs); complementar com testes unitarios, typecheck, build, curl/smoke HTTP e validacao manual guiada.
- Logs de producao ou erro real: diagnosticar pelo corpo da resposta/log, nao por suposicao.

## Referencias

Leia apenas a referencia necessaria para a tarefa:

- [references/seguranca.md](references/seguranca.md): invariantes de seguranca (auth/sessao, isolamento de tenant/IDOR, admin/escalacao, rate limit, webhooks e auth de maquina, segredos, criptografia, vazamento de erro, headers, blindagem de banco) e checklist. Ler antes de tocar auth, tenant, banco, pagamento, webhook, cron ou endpoint publico, e ao fazer varredura de seguranca.
- [references/operacao-versionamento.md](references/operacao-versionamento.md): versionamento, roadmap, test map, branch/release, docs obrigatorios e checkpoint.
- [references/readme-fonte-de-verdade.md](references/readme-fonte-de-verdade.md): como criar e atualizar o `README.md` raiz como fonte de verdade (atualizar sem reescrever, badges/visuais, integracoes e homologacoes, mapa de rotas, diagramas Mermaid mantidos e expandidos). Ler ao atualizar o README apos mudanca de versao, rota, integracao, stack, env ou estrutura.
- [references/arquitetura-testes.md](references/arquitetura-testes.md): stack, modulos, padroes de codigo, comandos, bancos, Prisma, GCP e runtime.
- [references/produto-ui.md](references/produto-ui.md): norte do produto, operacao, admin, storefront, mesa/comanda, WhatsApp e design system.
