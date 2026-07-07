# README.md raiz como fonte de verdade

O `README.md` raiz do `leaf-delivery` e a fonte de verdade mais consultada do sistema. Ele
descreve fielmente a estrutura, as rotas, as integracoes e o estado real do produto. Manter o
README fiel NAO e cosmetico: e o mapa que qualquer pessoa (socio, dev, agente, investidor) usa para
entender o sistema. Esta referencia define COMO criar e atualizar esse README.

## Regra central: atualizar, nunca reescrever do zero

- **Atualizar, nunca apagar e refazer.** Preservar os bons ossos: pitch, diagramas, governanca de
  mesa, fluxo PIX, licenca, familia Leaf. Editar de forma cirurgica, secao a secao.
- **Nao inventar conteudo.** Todo dado sai de fonte de verdade do repo (ver "Fontes a espelhar").
  Se algo nao existe, dizer que nao existe (ou "em breve"/"planejado"), nunca prometer.
- **Honestidade de status.** Distinguir sempre: `homologado` / `em producao` / `em breve` /
  `planejado`. Nao marcar como entregue o que so foi anunciado. Pendencia de go-live (conta, chave,
  env, homologacao) fica em `docs/aftersprints/after<versao>.md`, nao no README.
- **Surgical.** Nao reformatar, reindentar nem reordenar secoes que a mudanca nao afeta.

## Quando rodar

Sempre que o README divergir da realidade: mudanca de versao/live, rota nova (pagina ou API),
integracao/homologacao nova ou com status alterado, mudanca de stack/env/script, ou mudanca na
estrutura de pastas. Tambem ao fechar uma versao cujo escopo mexeu na superficie do produto.

## Enquadramento de versao

- Badge `Live` = o que esta DE FATO em producao (ultima versao mergeada e deployada). Uma badge
  secundaria cita a branch/sprint em andamento (ex.: `Sprint-1.8.15_em_andamento`).
- Nao antecipar no README uma versao que ainda nao foi cortada (bump + changelog + deploy). O bump
  de `package.json`/`CURRENT_APP_VERSION` e o corte de release, tarefa separada.

## Secoes que o README mantem fieis

1. **Cabecalho e badges.** Badges agrupadas por fileira tematica (Stack, Infra/observabilidade,
   Integracoes homologadas, Qualidade e em breve). Conferir versoes contra `package.json`. Linha de
   "capabilities" em emoji ajuda o scan. Expandir badges e visuais e desejavel, nao poluicao.
2. **Estado da Live.** Tabela por area com estado real (Entregue / Homologado / Em breve /
   Planejado). Sem "canario"/"LIVE READY local" congelado.
3. **Arquitetura Next + Nest.** Nest e o backend oficial (`apps/api` Fastify + `apps/worker`
   BullMQ). Next e frontend/BFF; rotas Next de negocio sao wrappers de compatibilidade; storefront
   publico e reescrito para o Nest por `src/proxy.ts`. Sem numeros/commits/revisions congelados de
   migracao antiga.
4. **Integracoes & Homologacoes.** Tabela com uso, modelo de negocio (DELIVERY/SHOWCASE/ambos) e
   status por integracao. Espelha `src/lib/integrations/plugin-catalog.ts`.
5. **Rotas & API.** Tabela de paginas (Next App Router) + mapa CURADO da API Nest por area
   (`public`/`internal`/`admin`/`webhooks`/`auth`+`health`). Nao esgotar os endpoints: apontar
   `docs/sprints/routes-migration.md` para o inventario completo.
6. **Stack, Estrutura, Scripts, Variaveis.** Alinhar com o repo real (incluir `apps/`, `src/proxy.ts`,
   `hooks`/`stores`/`workers`, Clerk no Auth, Upstash/BullMQ, Sentry, Storybook/Playwright). Env com
   nomes exatos (conferir por grep em `src`/`apps` e em `.env.example`).
7. **Roadmap e Documentos.** Mover para "entregue" o que ja saiu; manter futuro real. Linkar os docs
   vivos e conferir que cada link resolve.

## Diagramas Mermaid: manter, atualizar e EXPANDIR

Os Mermaids sao um ativo querido do README. Guard rails em `docs/product/mermaid.md` (workflow
`@mermaid.md`): **adiciona/atualiza, nunca recria**; cada diagrama tem um `<!-- mermaid:id=... -->`
estavel; reflete `roadmap.md` + `test.map.md`.

- **Manter** os existentes; corrigir so o que estiver factualmente errado.
- **Adicionar** novos para o que nasceu depois (ex.: arquitetura Next+Nest, fluxo do totem,
  integracoes por modelo, verificacao +18, governanca de notificacoes, monetizacao de ads). Cada um
  com seu proprio `<!-- mermaid:id=... -->`.
- **Paleta Leaf:** `#0b6632` (concluido/ativo), `#17201b` (dark/auth), `#f2a51a` (pendente/warning),
  `#a9363e` (bloqueado/erro), `#72a83c` (secundario), `#635BFF` (infra/integracao), `#f7f8f3`
  (superficie), `#6b7280` (planejado/em breve).
- **Failsafes:** validar a sintaxe mentalmente antes; quebrar linha com `\n` (nunca `<br>`); labels
  com `()`/`[]`/`&` sempre entre aspas; node IDs estaveis.

## Fontes a espelhar (nao inventar)

- `package.json`: stack, versoes e scripts.
- `src/lib/integrations/plugin-catalog.ts`: status e modelo de cada integracao.
- `src/lib/notifications/notification-visibility.ts` + `docs/product/notifications-governance.md`:
  governanca de notificacoes.
- `docs/sprints/routes-migration.md`: inventario e estado Next x Nest.
- `apps/api/src/**`: controllers/endpoints reais.
- `CHANGELOG.md`: features por versao.
- `docs/aftersprints/after*.md`: pendencias de go-live (o que fica FORA do README).
- `.env.example` + grep de `process.env.*` em `src`/`apps`: nomes exatos de env.

## Verificacao (entrega de doc)

1. Contagem de fences ``` par (bloco de codigo/mermaid fechado). Cada Mermaid da secao de diagramas
   com seu `mermaid:id`, paleta Leaf e sem `<br>`. Renderizar o preview (GitHub/VS Code) para
   confirmar que todos compilam.
2. Cada link interno `./...` resolve (checar por `ls`/`test -e`).
3. Cada rota de pagina citada existe em `src/app/**/page.tsx`; cada area de API em `apps/api/src/**`.
4. Grep por referencia stale (versao antiga, numeros congelados, "canario") deve voltar vazio.
5. E doc: sem gate de codigo. Registrar no fechamento; se contar como entrega da versao, refletir em
   `roadmap.md`/checklist.
