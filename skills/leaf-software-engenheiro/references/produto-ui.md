# Produto, Operacao e UI

## Norte do produto

Leaf Delivery e um SaaS operacional para restaurante/delivery proprio:

- catalogo publico por tenant;
- checkout delivery, retirada e salao;
- pedido indo para WhatsApp/anotacao operacional;
- status, tracking publico e historico do cliente;
- tenant admin para operacao diaria;
- master/platform admin para rede, tenants, modulos e BI;
- mesa/comanda/garcom/KDS como extensao do mesmo core de pedido.

Nao transformar em ERP completo antes do necessario. Priorizar o loop operacional: pedido -> WhatsApp/anotacao -> status -> tracking -> cozinha/mesa quando aplicavel.

## Fluxos principais

Cliente:

- `/` landing publica quando host nao resolve tenant.
- `/demo` demo guiada.
- `/store/[tenantSlug]` catalogo.
- `/store/[tenantSlug]/track/[token]` tracking.
- `/track/[token]` tracking por host/atalho.
- `/store/[tenantSlug]/table/[tableToken]` QR de mesa.

Admin:

- `/platform-admin/login`
- `/platform-admin`
- `/tenant-admin/[tenantSlug]`
- `/tenant-admin/[tenantSlug]/staff-login`
- `/_admin` via host de tenant quando aplicavel.

APIs criticas:

- `/api/public/orders`
- `/api/public/quote`
- `/api/admin/orders`
- `/api/admin/tables`
- `/api/admin/tables/bulk`
- `/api/internal/outbox/process`
- `/api/webhooks/abacatepay`
- `/api/webhooks/whatsapp/[provider]`

## Mesa, comanda, garcom e cozinha

- Mesa/comanda deve reutilizar o modelo completo do delivery: variantes, adicionais, combos, meio a meio e precificacao por filial.
- Configuracao de mesa deve favorecer lote quando o usuario reclamar de configurar uma por uma.
- `capacity` deve refletir cadeiras visiveis.
- `ComandaModal` e a visao de Pedidos devem abrir a mesma leitura operacional para pedidos `DINE_IN` com mesa.
- Funcionario operacional deve ver o que precisa para operar; QR/setup/link/impressao ficam restritos a admin/master quando aplicavel.
- KDS/cozinha deve ordenar por urgencia, mostrar contexto de delivery/retirada/mesa e permitir impressao termica quando o fluxo exigir.

## WhatsApp, PIX e operacao ao vivo

- Pedido novo em banco deve drenar o evento de outbox/notificacao necessaria sem depender apenas de botao manual para a primeira mensagem.
- Mensagens manuais devem processar lote suficiente para nao ficarem presas atras de notificacoes antigas.
- Tracking publico deve ter polling/no-store, feedback visual continuo e acoes de WhatsApp com cooldown quando houver problema.
- PIX real usa Abacate Pay, webhook `transparent.completed`, persistencia idempotente e `ENABLE_PIX_CHECKOUT=true` em runtime.
- Tenant demo pode retornar PIX fake/homologacao sem chamar integracao real.

## Design system

Arquivos fonte:

- `DESIGN.MD`: indice visual e relacao admin/cliente.
- `COLORS.MD`: paleta obrigatoria.
- `ADMIN.DESIGN.md`: painel operacional.
- `CLIENT.DESIGN.md`: storefront, tracking e jornada do cliente.
- `reforma.md`: decisoes/refinamentos de UI em andamento.

Paleta oficial:

- `#D4960F` Harvest Gold
- `#429C01` Forest Green
- `#FF8248` Coral Glow
- `#810000` Maroon
- `#FFEF93` Light Gold
- `#E8EFF7` Alice Blue
- `#8976D7` Soft Periwinkle
- `#111213` Onyx
- `#2D1A01` Dark Coffee

Nao criar cores fora da paleta para background, CTA, badge, icone, borda, grafico, ilustracao ou estado.

## Admin UI

- Painel denso, operacional e escalavel; nao parecer landing page.
- Base branca/off-white, divisorias claras, sombras minimas.
- Busca, filtros e acoes no topo operacional.
- Listas e tabelas compactas por padrao.
- Entidades editaveis devem abrir edicao ao clicar no card/linha; lapis discreto serve como pista.
- Widgets de configuracao podem ser recolhiveis e lembrar estado.
- Mobile do tenant admin pode ter launcher operacional dedicado para Pedidos, Cozinha e Mesas.
- Nada de gradientes estruturais, blocos glass, hero decorativo ou cards gigantes sempre abertos.

## Cliente UI

- Mobile-first, comercial e orientada a conversao.
- Produto, foto, preco, CTA, tempo e disponibilidade precisam aparecer rapido.
- Storefront nao deve parecer painel administrativo.
- Carrinho/checkout em drawer, com leitura imediata e sem prender o cliente em formulario longo.
- Tracking deve priorizar status, ETA, mapa/rota quando houver e volta ao catalogo sem competir com o status.
- Landing publica deve ser clara, usar prints reais em `public/showcase` e manter Leaf como primeiro sinal visual.

## Copia e tom

- Evitar copy generica de IA, promessas infladas e jargao.
- Falar como operador/produto real: pedido, cozinha, comanda, WhatsApp, fila, acompanhamento, filial, mesa.
- Em docs e respostas, registrar estado real: validado, parcial, pendente ou bloqueado.
