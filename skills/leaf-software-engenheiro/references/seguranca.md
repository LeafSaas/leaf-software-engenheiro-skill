# Seguranca (Leaf)

Regras inegociaveis de seguranca para projetos Leaf. Portado do devsec do
leaf-delivery (`docs/devsec/`) e generalizado para as duas stacks de auth em uso:

- **Sessao custom** (leaf-delivery): JWT HS256 + `adminAuthSession`, guards Nest.
- **Clerk** (leafdfe/dfeknowledge, L-emissor): `auth()`/`currentUser()` do Clerk +
  Prisma. Aqui o "token" e o do Clerk; a autorizacao de dominio (plano, role,
  tenant/empresa, admin) e SEMPRE relida do banco, nunca do claim do cliente.

Codigo novo que quebrar qualquer invariante abaixo NAO entra. Cada regra tem "a
regra" e "como fazemos aqui". Principio central: **seguranca nao depende de
segredo do schema**. Assuma que o atacante conhece toda tabela, coluna e rota. A
blindagem real e a lista abaixo.

---

## 1. Autenticacao e sessao

1. **O token/claim nunca e a fonte de verdade de autorizacao.** `role`, `plano`,
   `isAdmin`, `tenantId`/`companyId` e permissoes sao **relidos do banco a cada
   request** a partir do `userId` autenticado. No Clerk: `auth()` da o `userId`;
   nunca confiar em `sessionClaims.metadata.role`/`publicMetadata` para DECIDIR
   acesso (é advisory, editavel por caminhos administrativos, pode estar stale).
   Reler o `User`/membership do Prisma e decidir a partir dai.
2. **Sessao fail-closed.** Sem usuario autenticado resolvido no servidor, o
   request e rejeitado (401/403). Nunca "assume anonimo = ok" num handler que
   muta ou le dado de usuario.
3. **Segredos de sessao/assinatura obrigatorios em producao.** App quebra no boot
   se faltar (nao roda com fallback hardcoded). Fallback de dev so vale fora de
   producao e nunca pode existir em ambiente exposto.
4. **Cookies proprios** (quando houver): `httpOnly`, `secure` em prod,
   `sameSite=lax`, `path=/`, TTL absoluto + idle. Manter esses flags em qualquer
   novo ponto que emita cookie de sessao.
5. **CSRF:** mutacoes (POST/PUT/PATCH/DELETE) que agem por cookie passam por
   checagem de same-origin. Endpoint novo que muta estado usa o guard. No App
   Router, rotas autenticadas por Clerk cookie tambem precisam de same-origin
   nas mutacoes; nao confiar so no fato de estar logado.

**Regra para endpoint novo:** guard de sessao (existe usuario?) + guard de
escopo (esse usuario pode ESTE recurso?). O guard de sessao sozinho NAO valida
tenant nem permissao.

---

## 2. Isolamento de tenant/owner (anti-IDOR) - a falha #1 de codigo gerado por LLM

**Invariantes:**

1. **`tenantId`/`companyId`/`userId` de escopo nunca vem do request como fonte de
   autorizacao.** Deriva do principal autenticado no servidor. Um id de recurso
   na URL (`[id]`, `[companyId]`, `[cnpj]`) e input nao confiavel.
2. **Toda query que recebe um `id` do request DEVE tambem filtrar pelo dono.**
   Padrao correto: `findFirst({ where: { id, userId } })` (ou `companyId in
   (empresas do usuario)`) ANTES de `update`/`delete`/download. Nunca
   `findUnique({ where: { id } })` seguido de acao, sem checar o dono.
3. **Defesa em profundidade no repositorio.** Preferir escopo por dono dentro do
   proprio repositorio/lib, para que um guard esquecido no handler nao vire
   vazamento cross-tenant. Nao dependa so de uma checagem no topo do route.
4. **Nunca passar escopo vazio** (`tenantSlug: ""`, `where: {}`, `companyId:
   undefined`) para um handler: isso zera o filtro e retorna dado de todo mundo.
5. **REST API v1 / API key:** a key resolve um tenant/usuario fixo; TODA query da
   API escopa por esse tenant. Um id de documento de outro tenant retorna 404.

**Teste obrigatorio para endpoint com `:id`:** usuario A pedindo `:id` do usuario
B recebe 403/404, nunca o dado. Escrever esse teste junto com o endpoint.

---

## 3. Admin / conta de plataforma

1. Admin = **allowlist de email** (`PLATFORM_ADMIN_EMAILS`) que **falha fechada**:
   allowlist vazia/ausente => ninguem e admin. Cuidado com `"".split(",")` que
   gera `[""]` e pode dar match com email vazio/undefined: normalizar, filtrar
   vazios, e exigir email verificado nao-vazio.
2. Autorizacao **server-side em toda rota admin** (a rota relê o email do usuario
   do Clerk/DB e cruza com a allowlist). A UI e o middleware NUNCA sao a
   fronteira; middleware/matcher pode ter regex furada e ser bypassavel. Cada
   handler admin revalida.
3. **Nenhum campo controlado pelo usuario decide privilegio.** `role`, `plano`,
   `isAdmin`, `premium`, `subscriptionStatus` so mudam por caminho server-side
   autorizado, nunca a partir de `body.role`/`body.plan`/claim do cliente.
   Endpoints tipo `switch-role`, `grant-*-premium`, `switch-plan`, `upgrade` sao
   os primeiros a auditar: quem paga? quem autoriza? o usuario pode se dar plano
   pago ou premium sem pagamento/admin?
4. Hashing de senha (quando houver): **bcrypt cost >= 12**, `compare` timing-safe.
   Nunca comparar segredo com `===`.
5. **Backdoor de dev nunca chega em producao.** Qualquer acesso `beta`/dev exige
   `NODE_ENV=development` (+ flags) e essas envs jamais existem no runtime de
   prod. Garantir ausencia no Cloud Run/Vercel.

---

## 4. Rate limit e bruteforce

1. Endpoints sensiveis (login, reset, 2FA, cupom, pagamento, envio de email,
   criacao de lead/telemetria publica, aceitar convite) tem `checkRateLimit` com
   namespace proprio e discriminador adequado (email/IP/tenant/sessao).
2. Em producao usar Upstash (distribuido). O fallback em memoria e por-processo e
   NAO protege multi-instancia: `UPSTASH_REDIS_REST_URL/TOKEN` devem estar
   setados em prod. Nunca confiar so no fallback em ambiente exposto.
3. IP so e confiavel com proxy trust configurado; sem isso, keyar por
   usuario/sessao, nao por IP forjavel.

---

## 5. Webhooks e auth de maquina

1. **Webhook de terceiro so e aceito com ASSINATURA verificada** antes de agir:
   Clerk usa `svix` (`Webhook(secret).verify(payload, headers)` com
   `svix-id/svix-timestamp/svix-signature`); gateways de pagamento
   (Abacatepay/Asaas) e Chatwoot usam HMAC/segredo proprio. Sem assinatura valida
   => 400/401, nunca processa o body. Nunca "if (!secret) skip verification".
2. **CRON e jobs internos** (`CRON_SECRET`, `INTERNAL_JOB_SECRET`): comparar com
   `crypto.timingSafeEqual`, nunca `===`. Sem fallback default que valha em prod
   (`process.env.X || "dev"` que aceita o default e furo). Falta do segredo =>
   endpoint fecha.
3. **API key da REST:** guardar HASH (nao plaintext), lookup que nao vaza timing,
   scopes aplicados por rota, tenant do key escopando toda query, rate limit.
   Nunca uma "chave global via env" que da acesso total sem tenant.

---

## 6. Segredos

1. **Nenhum segredo no codigo nem em arquivo rastreado.** `.env*` gitignored.
   Producao usa Secret Manager / env do runtime (`DATABASE_URL`, `CLERK_*`,
   `CERTIFICATE_MASTER_KEY`, `*_WEBHOOK_SECRET`, Upstash, `REST_API_KEY`).
   Conferir com `git ls-files | grep -iE '\.env'`: se um `.env` com segredo real
   estiver rastreado, e CRITICO (rotacionar a chave imediatamente).
2. Segredo live em `.env` de dev e risco real por disco/backup. Nao guardar
   segredo de producao em `.env` de maquina de dev; se aconteceu, **rotacionar**.
3. Comparacao de segredo de automacao/webhook: sempre `timingSafeEqual`.
4. Nunca logar token, segredo, connection string ou corpo cru com dado sensivel.
   Logar presenca (`Boolean(secret)`), nunca o valor.

---

## 7. Criptografia de dado sensivel (certificados A1, senhas)

1. Dado sensivel em repouso (certificado A1 `.pfx`, senha do certificado) cifrado
   com algoritmo **autenticado**: AES-256-GCM, **IV aleatorio por operacao**,
   tag de autenticacao verificada na decifra. Nunca ECB, nunca IV fixo/reusado,
   nunca "cifra" que e so base64/xor.
2. `CERTIFICATE_MASTER_KEY` >= 32 bytes, sem fallback default. Chave derivada de
   senha usa KDF com sal (scrypt/PBKDF2), nao hash cru de string curta.
3. Material do certificado nunca volta ao cliente nem aparece em log.

---

## 8. Erros e vazamento de informacao

1. **Nunca devolver `error.message`/`error.meta` cru ao cliente** sem gate de
   ambiente. Em producao o cliente recebe mensagem generica; detalhe vai para
   log interno/Sentry. Erros do Prisma vazam nome de tabela/coluna/constraint via
   `error.meta` e `error.message`: traduzir para codigo de dominio
   (`P2002` => "registro ja existe" sem citar a coluna; `P2025` => 404 generico;
   `P2003` => "operacao invalida").
2. Rotas de **debug/diagnostico** (`/api/debug/*`) nunca expostas sem auth em
   producao (idealmente removidas do bundle de prod). Uma rota que dumpa Prisma,
   env ou schema e vazamento direto.
3. Health endpoints enxutos: nao expor detalhe de DB, schema, env nem versao
   sensivel a anonimo.

---

## 9. Headers e transporte

Mantidos em `next.config.ts`: CSP, HSTS (`max-age=31536000; includeSubDomains`),
`X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`,
`Permissions-Policy`. Meta: remover `'unsafe-inline'`/`'unsafe-eval'` do
`script-src` (nonce/hash). CORS nunca `*` com credenciais; refletir origem so da
allowlist.

---

## 10. Blindagem do banco (Postgres/Prisma)

1. Banco **sem IP publico** / so private IP ou proxy + IAM. Credencial so no
   Secret Manager, com rotacao. Sem `authorized-network` aberta (`0.0.0.0/0`).
2. **Menor privilegio (meta):** role de migration (DDL) separada da role de
   runtime (DML apenas, sem DDL/owner). Runtime nao deveria rodar `migrate
   deploy` com role de owner.
3. **RLS (meta, defesa em profundidade):** habilitar `ROW LEVEL SECURITY` nas
   tabelas com escopo de tenant/usuario, com policy amarrando ao principal do
   request. Transforma um `where` esquecido em "zero linhas" em vez de
   vazamento. Por fases, comecando pelas tabelas mais sensiveis.
4. Todo `$queryRaw` de runtime **parametrizado** (tagged template / `Prisma.sql`).
   `queryRawUnsafe`/`executeRawUnsafe` so em script offline, nunca em handler de
   request.

---

## Checklist de revisao de seguranca

Toda entrega que toca auth/tenant/banco/pagamento/endpoint publico:

- [ ] Endpoint com guard de sessao + guard de escopo (dono/tenant) correto.
- [ ] Query com `:id` do request filtra pelo dono (`userId`/`companyId`/tenant).
      Nenhum `findUnique(id)` seguido de acao sem checar dono. Nenhum escopo vazio.
- [ ] Nenhum `role`/`plano`/`isAdmin` lido do claim/body para decidir acesso
      (reler do DB). Rotas `switch-role`/`grant-premium`/`switch-plan` auditadas.
- [ ] Rota admin revalida a allowlist no servidor (nao so middleware). Allowlist
      falha fechada.
- [ ] Webhook verifica assinatura antes de agir. CRON/job/api-key com
      `timingSafeEqual`, sem fallback de segredo.
- [ ] Mutacao por cookie protegida por same-origin/CSRF.
- [ ] Nenhum `catch` devolvendo `error.message`/`error.meta` cru ao cliente.
      Nenhuma rota `/api/debug/*` sem auth em prod.
- [ ] Nenhum segredo novo no codigo; `.env` fora do git. Segredo de automacao
      comparado timing-safe.
- [ ] Dado sensivel (certificado A1) cifrado com AES-GCM + IV por operacao.
- [ ] Endpoint sensivel com rate limit e namespace proprio.
- [ ] Teste de acesso cross-tenant (A no `:id` de B => 403/404).
- [ ] `npm run typecheck` e `npm test` verdes.

---

## Como auditar (recon read-only)

Varredura de seguranca segue o metodo do devsec do delivery: recon read-only +
leitura direta dos arquivos criticos, sem alterar producao. Particionar por
superficie (IDOR/tenant, admin/escalacao, webhooks/maquina, vazamento/segredos)
e rodar frentes independentes. Cada achado vira linha em `docs/devsec/audit-*.md`
com ID, severidade, arquivo:linha, cenario de exploracao, evidencia e remediacao.
Marcar CONFIRMADO vs SUSPEITO; nao reportar suspeita como fato. Rodar Q.A.
adversarial (tentar refutar cada achado) antes de fechar.

O projeto mantem seus padroes vivos em `docs/devsec/` (pasta LOCAL, gitignored,
nao commitar): `security-standards.md`, `db-blindagem.md`, `audit-<data>.md`.
