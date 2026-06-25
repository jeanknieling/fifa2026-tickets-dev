# Quartas de Final (F2) — Kickoff & Handoff

> **Comece por aqui na próxima sessão.** Este doc é o ponto de partida para construir o lab **Quartas de Final** (F2: Gateway YARP + Identidade do cliente). Gerado em 2026-06-25 ao fim de uma sessão longa.

---

## 1. O que está PRONTO (Oitavas / F1 + base)

- **Story 2.10 (compra multi-item · fan-out)** — `Done` (QA CONCERNS resolvido). 1 POST do carrinho → N mensagens no Service Bus → N consumidores → N gravações. Backward-compat (single shape) preservado. Sem migration.
- **Deployado e validado no HML** (`tickets.tfteccloudlabs.cloud`): smoke e2e completo pela UI (orderId + 2 correlationIds + recibo), Service Bus (active=0/DLQ/Incoming=Outgoing), App Insights (1 entry → 2 consumers → 2 status + dependencies SQL/Service Bus).
- **Fixes da sessão:** `/matches` (VITE_API_URL=/api relativo p/ backend privado), App Insights worker logs (remoção do filtro default no `Program.cs`), Login v2 escondido quando Entra não configurado.
- **Diagramas draw.io (ícones Azure reais):** `docs/diagrams/oitavas-f1-multi-item-fanout.drawio` (fan-out animado, passos ①→⑥) e `docs/diagrams/arquitetura-geral-fifa2026.drawio` (Modernização + Oitavas — Quartas ainda NÃO desenhada). PNGs em `docs/images/`.
- **Brainstorming de identidade:** `docs/research/2026-06-25-identidade-external-id-quartas.md`.
- Tudo no **fork `origin`** (tftec-guilherme). Último commit: `373207d`.

## 2. Pendências OPERACIONAIS imediatas (próxima sessão)

- **PUSH TFTEC (`upstream`)** — o deploy/push pro TFTEC foi reservado para "amanhã" (2026-06-26). A `main` do fork está à frente do upstream com TODO o trabalho das Oitavas + multi-item + fixes + diagramas. **Primeira tarefa: @devops leva isso pro upstream** (owner quer "só a branch"; confirmar antes). Exclusivo @devops, gate `AIOX_ACTIVE_AGENT=devops`.
- (Opcional) atualizar o **PPT das Oitavas** (slide 8/12/16) que está pré-multi-item — ver `[[ppt-oitavas-and-quartas-direction]]`.

## 3. Quartas (F2) — ESCOPO e DECISÕES do owner

Lab combina **Gateway YARP (plano de dados)** + **Identidade do cliente (plano de identidade)**.

| Dimensão | Decisão (owner, 2026-06-25) |
|----------|------------------------------|
| **Cliente (B2C)** | Microsoft **Entra External ID (CIAM)** com **social login Google** pré-configurado pelo instrutor (email+OTP fallback) |
| **Tenant CIAM** | **Trial 30 dias** (sem subscription/cartão, recriável) |
| **Admin (B2B)** | **CONSTRUIR** o login de admin: Entra ID **workforce** + **App Roles** (não só conceito) |
| **Migração v1→CIAM** | **Passo prático do lab** (executar a migração de `users` v1 → CIAM) |
| **Gateway** | Gateway YARP: proxy, rate-limit (5/min→429), output cache (30s), CORS, **validação de JWT** do CIAM |

**Pontos-chave técnicos (do memo + ADE-005):**
- Authority muda de `login.microsoftonline.com` (workforce) → **`<tenant>.ciamlogin.com`** (CIAM) no `authV2.ts` e no gateway `Program.cs` (issuer/audience).
- O encadeamento **Gateway → Function → SQL** (`oid` → `X-Entra-OID` → `entra_oid`) é **issuer-agnóstico** — NÃO muda.
- `entra_oid` **COEXISTE** com `users` v1 (bcrypt). O contraste v1 vs v2 É o produto didático.
- B2C legado (Azure AD B2C) está **depreciado** — não usar.

## 4. ⚠️ DECISÃO PENDENTE do owner (resolver no início da próxima sessão)

O owner escolheu as **2 opções mais pesadas** (admin + migração hands-on). Com Gateway + 2 mundos de identidade + migração, **provavelmente passa de 6h**. **Pergunta aberta:** dividir **Quartas em 2 sessões**?
- Proposta: **Quartas-A** = Gateway YARP + cliente External ID (CIAM). **Quartas-B** = admin (workforce + App Roles) + migração v1→CIAM.
- (Owner ainda não bateu o martelo nessa divisão.)

## 5. PLANO DE EXECUÇÃO (passo a passo)

1. **@architect** — formalizar **nova ADE** (`docs/architecture/ade-00X-identity-external-id.md`) que **supersede a ADE-005**, registrando as 4 decisões acima + invariantes + trade-offs. (Proposta de ADE em bullets no §7 do memo.)
2. **@pm** — atualizar blueprint (`docs/workshops/2026-blueprint-living-lab-azure.md`, F3) + epic (`docs/epics/EPIC-002-*`, stack/riscos #2/#6).
3. **@sm** — story(ies) das Quartas (ou 2 stories se dividir A/B).
4. **@dev/@ux** — build dos artefatos no padrão `docs/workshops/phase-02/` (já existe o set do Gateway):
   - `slides.md` (Reveal.js), `SPEAKER-NOTES.md`, `README.md`, `PORTAL-GUIDE.md`, `intro-video-script.md`
   - novo **runbook** `docs/runbooks/aula2-f2-portal-guide-...md` (espelhar o do F1: `aula1-f1-portal-guide-ambiente-hml-cin.md`)
   - **desenho draw.io das Quartas** (Gateway + Identidade) — adicionar à arquitetura geral quando construído.
5. **Código (quando build):** `authV2.ts` (authority ciamlogin.com), gateway `Program.cs` (issuer/aud), `vite-env.d.ts` (vars), App Settings.

## 6. REFERÊNCIAS

- Memo identidade: `docs/research/2026-06-25-identidade-external-id-quartas.md`
- ADEs: `docs/architecture/ade-004-gateway-yarp.md` (F2 gateway), `ade-005-identity-easy-auth.md` (a superseder)
- F2 artefatos existentes: `docs/workshops/phase-02/` (slides/notes/README/PORTAL-GUIDE)
- F2 story: `docs/stories/2.2.story.md` (14 ACs, Done). F3 story: `docs/stories/2.3.story.md`
- Runbook padrão (F1): `docs/runbooks/aula1-f1-portal-guide-ambiente-hml-cin.md`
- Gateway code: `src/Fifa2026.V2.Gateway/` · Frontend MSAL: `Lovable/World Cup Tickets Hub/src/lib/authV2.ts`, `LoginV2Button.tsx`
- Desenhos do Rapha (estilo): OneDrive `…\07. Técnico\03 - 16 avos de final\…\arquitetura-fifa2026-tickets-paas.drawio` (página "Versão ícones Azure" usa os mesmos stencils azure2)

## 7. CONSTRAINTS

- **Fork only** até o owner liberar o push pro TFTEC.
- Push/PR/MCP/deploy = **@devops** exclusivo (gate `AIOX_ACTIVE_AGENT=devops`).
- Git Bash: `MSYS_NO_PATHCONV=1` em `az` com resource ids.
- HML privado: backend e SQL com `publicNetworkAccess=Disabled`; app só pelo **custom domain** `tickets.tfteccloudlabs.cloud` (URL nativa quebra auth/CORS).
