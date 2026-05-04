# Arquitetura, Desenvolvimento e Testes

## Stack atual

Base `leaf-delivery`:

- Next.js `16.2.4` e React `19.2.5`.
- TypeScript `5`, Tailwind `4`, ESLint `9`, Vitest `4`.
- Prisma `5.22` com PostgreSQL.
- UI com Shadcn/Radix, componentes em `src/components/ui`, `lucide-react`, `sonner`, TanStack Table.
- Deploy em Cloud Run + Cloud SQL + Cloud Build + Artifact Registry.
- Evolution API em VM separada; Scheduler chama outbox interna.

Antes de escrever codigo Next.js 16, leia a documentacao local relevante:

```bash
find node_modules/next/dist/docs -maxdepth 2 -type f | sort
```

Use os guias locais para APIs novas/deprecadas, App Router, middleware/proxy, metadata, cache, routing e config.

## Estrutura de software

Diretorios centrais:

- `src/app`: rotas publicas, admin, APIs e webhooks.
- `src/components/admin`: painel tenant/master e componentes operacionais.
- `src/components/admin/admin-console`: partes extraidas do console admin.
- `src/components/store`: storefront, checkout, tracking e componentes do cliente.
- `src/components/ui`: base visual reutilizavel.
- `src/lib/contracts`: contratos tipados de dominio/API.
- `src/lib/domain`: regras puras.
- `src/lib/repository`: repositorios por dominio, demo/prisma e integracoes de persistencia.
- `prisma`: schema, migrations e seeds.
- `tests` e `src/lib/__tests__`: suites Vitest.

Dominios alvo do monolito modular:

- `catalog`
- `table`
- `order`
- `payment`
- `kitchen`
- `customer`
- `notification`
- `reporting`

Preserve contratos publicos dos barrels existentes. Ao extrair, mantenha imports externos funcionando e cubra exports relevantes.

## Padroes de implementacao

- Preferir funcoes puras e contratos Zod/TypeScript para entradas publicas.
- Aplicar filtro por `tenantId`, `branchId` e permissao no banco quando possivel, nao em memoria.
- Evitar N+1 em rotas de platform/admin; usar `select`, `_count`, `groupBy` ou SQL agregado quando necessario.
- Para polling, usar cursor de servidor quando disponivel e proteger contra respostas antigas.
- Para imagens dinamicas de tenant/produto, manter allowlist/proxy fechado e `unoptimized` quando a URL vem de cadastro.
- Para outbox/inbox, manter consumidores idempotentes e eventos versionados.
- Para WhatsApp, tratar Evolution como integracao falivel; validar resposta real e fila.
- Para admin, mutacoes devem preservar contexto da tela; evitar reload duro.
- Para storefront, nao reescrever o shell inteiro sem auditoria previa.

## Comandos de validacao

Gates comuns:

```bash
npm run typecheck
npm test
npm run lint
npm run build
```

Banco local:

```bash
npm run db:up
npm run db:push
npm run db:seed:test-stores
npm run db:seed:scenario:tokyo
```

WhatsApp local:

```bash
cp .env.evolution.example .env.evolution
npm run wa:up
npm run wa:logs
```

Prisma/producao:

```bash
npx prisma validate
npm run db:prod:migrate:status
npm run db:prod:migrate:deploy
```

## Modos de teste

Modo demo:

- `LEAF_DELIVERY_DEMO_MODE=true`
- `npm run dev`
- Usar para UX, fluxo e painel sem preparar banco.

Banco local real:

- `npm run db:up`
- `npm run db:push`
- `.env.local` com `DATABASE_URL=postgresql://postgres:postgres@localhost:5433/leaf_delivery?schema=public`
- `LEAF_DELIVERY_DEMO_MODE=false`
- Seed com `npm run db:seed:test-stores`.

Cenario operacional Tokyo:

- Rodar `npm run db:seed:scenario:tokyo` apos o seed de lojas.
- Usar para BI, estoque, producao/receitas, mesas, pedidos e pico operacional.

Producao GCP:

- App: `https://leafdelivery.com.br`
- Tenants: `https://aurora-pizza.leafdelivery.com.br`, `https://tokyo-express.leafdelivery.com.br`, `https://adega-orbita.leafdelivery.com.br`
- Fallback tecnico: `https://leaf-delivery-rnrpsjwlhq-rj.a.run.app`
- Cloud Scheduler: `leaf-delivery-outbox-process`
- Project: `leaf-delivery-494212`, region `southamerica-east1`.

## Restricoes

- Nao usar Playwright neste projeto salvo se o usuario revogar explicitamente a regra corrente.
- Para storefront grande, evitar scripts Python auxiliares; editar de forma guiada e incremental.
- Nao antecipar K8s, Redis, RabbitMQ, SSE ou Pub/Sub como backlog ativo sem pedido e metrica real.
- Nao tratar `bis_skin_checked` como bug do app sem validar em navegador limpo.
