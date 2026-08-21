# Bootstrap — o passo-a-passo de criar um projeto novo com o piso do dia 0

> Este reference é a **receita executável** do `/scaffold-new`: a ordem de instanciar um projeto
> da casa, do workspace vazio ao overdev ligado. É o que o **`/scaffold-new`** executa por
> baixo. Cada passo aponta o piso que materializa (`references/piso.md`) e a estrutura que segue
> (`references/estrutura.md`). **Nada é opcional; nada é "pra depois".**

## Fase 0 — Decidir antes de criar (plan-first)

Scaffoldar sem decidir a fronteira é montar um monólito por acidente. Antes de `mkdir`:

1. **Nome e slug do projeto** (`<projeto>`) — snake_case, o prefixo de todos os repos.
2. **Bounded contexts** — quais aplicações o produto tem (o core do negócio + o que mais). Cada
   um vira um repo `<projeto>_<ctx>_<lang>`. Na dúvida, comece com **um** contexto de produto — é
   melhor um contexto bem-fechado que um monólito com dois domínios colados.
3. **Linguagem por serviço + ADR inicial** (`references/linguagem.md`): fit → ADR `accepted`. O
   auth tende a Rust; o core segue o fit; o front é Node.
4. **Domínio** (`<domain>`) — pra `auth.<domain>` e o roteamento.
5. **Grave o plano** em `<projeto>_archive/plan/<data>-scaffold.md` (topologia + ADRs + o
   checklist de bootstrap abaixo) e **peça aprovação** se a topologia for grande. Este é um alvo
   natural de `/eng-overdev` (o piso vira um checklist exaustivo que só fecha provado).

## Fase 1 — O workspace e o archive

6. **Confirme a contenção:** o diretório atual é o workspace; tudo nasce **dentro** dele
   (`references/estrutura.md` §1). Nada de `cd ..`.
6.1. **Copie os templates do dia 0** — `assets/templates/gitignore.tpl` → `.gitignore` (em **cada**
   repo) e `assets/templates/env.example.tpl` → `.env.example`. O `.gitignore` entra **antes do
   primeiro commit**: segredo commitado uma vez está **vazado para sempre** (a ação é rotacionar,
   não "remover o commit"). O `.env.example` já vem com as **4 camadas** do piso de efeito externo
   preenchidas (`MAIL_PROVIDER=sink`, `MAIL_SINK_API`, `TEST_MAIL_DOMAIN`, `MAIL_MAX_PER_RUN`).

7. **Crie `<projeto>_archive/`** já no dia 0, com a planta CANÔNICA da `schematize-archive`
   (`references/archive.md`, seção "Estrutura") — ou rode `/archive-init`, que a materializa.
   Não invente a árvore aqui: planta duplicada foi o achado B2 da vistoria. O rastro existe antes
   do código (`references/piso.md` §7).

## Fase 2 — O auth, PRIMEIRO (app separada)

O IAM é dia 0 e é o coração do piso — então vem cedo, não por último.

8. **`<projeto>_auth_<lang>`** — o IdP: microserviço próprio, servido em `auth.<domain>`, com o
   **baseline 2FA** (ID≠email, senha argon2id + Email OTP, passkey núcleo, ReBAC multi-tenant,
   deny-by-default, OIDC/OAuth2.1+PKCE, chave só aqui + JWKS). Use o `/<slug>-iam` da linguagem
   escolhida pra scaffoldar (`references/piso.md` §3, `schematize-engineering/references/iam.md`).
9. **`<projeto>_authfront`** — o front de login/registro/recuperação/gestão de fatores
   (`schematize-web`; segredo server-side).
10. **Fronteira de delegação:** o app principal (Fase 3) já nasce **cliente OIDC** do auth —
    nunca com login próprio embutido.

## Fase 3 — Os contextos do produto + o front

11. **`<projeto>_<ctx>_<lang>`** (1..N) — os bounded contexts do produto, cada um com: segurança
    (segredo fora do cliente, query parametrizada, auth server-side), estrutura de testes+pentest,
    observabilidade integrada, **delegação ao auth por OIDC**, e **independência de runtime**
    (sobe sozinho, degradação graciosa — `references/estrutura.md` §7).
12. **`<projeto>_front`** — o front do produto (`schematize-web`), separado do backend.
13. **Root limpo** em cada repo: código, config, `README`, `CLAUDE.md` (rode `/<slug>-claude`),
    `LICENSE`. Todo MD gerado vai pro archive.

## Fase 4 — O ops (control plane)

14. **`<projeto>_ops`** — a interface única (`references/piso.md` §4): comandos idempotentes
    (`bootstrap`/`install`/`redeploy`/`update`/`config`/`migrate`/`health`/`rollback`/`logs`/
    `reset`/`test`), **fluxo de promoção** (dev→teste→git→hml→prd), **deploy destrutivo por seed**
    (`/<app>/.env`), **isolamento por app** (user Linux + systemd hardened por serviço),
    **instalação paralela** (`nproc`). Use o `/<slug>-ops` da linguagem. O `bootstrap` do ops é o
    que clona os repos irmãos em `/<app>/`.

## Fase 5 — CI, DoD e observabilidade amarrados

15. **CI com deploy gated:** pipeline que roda o test kit + gate da DoD e **só promove o que
    passou**; promoção pra hml/prd gated pelo fluxo (`references/piso.md` §4, §6).
16. **DoD (§35) como gate de PR/entrega** em cada repo (review + testes + archive + índice).
17. **Observabilidade** integrada em todo serviço e no ops (health/logs/métricas/tracing —
    `references/piso.md` §5).
18. **Efeito externo em SINK por default de fábrica** (`references/piso.md` §9): o `.env.example`
    e o seed do ops já nascem com `APP_ENV`, **`MAIL_PROVIDER=sink`**, `TEST_MAIL_DOMAIN=test.<domain>`
    e **`MAIL_MAX_PER_RUN=50`**; o `docker-compose` de dev já sobe o **Mailpit** (UI/API em `:8025`,
    SMTP em `:1025`); o guard mora **dentro do provider** e vem com o **teste que vê a recusa**;
    fixtures/seeds só emitem `<papel>+<run-id>-<n>@test.<domain>`; o gate
    `check-external-effects.sh` entra no CI e o **DNS em rota nula** (null MX + SPF `-all` +
    DMARC `p=reject`) abre como item pra `schematize-infra`.
19. **Índice/MAPA** (§39) gerado (`/eng-index`) e no archive.

## Fase 6 — Ligar o overdev e fechar provado

20. **Overdev disponível** (`/eng-overdev`) — e, se o bootstrap rodou como overdev, o checklist do
    piso só fecha **verificado, não auto-declarado**.
21. **Rode o `/scaffold-check`** contra o próprio projeto recém-criado: ele prova que o piso do
    dia 0 está **de fato** lá (`references/check.md`). Bootstrap só termina com **zero itens do
    piso faltando**.

## Checklist de bootstrap (derivável direto pro overdev)

```
- [ ] Fase 0: slug, bounded contexts, linguagem+ADR por serviço, domínio, plano no archive
- [ ] <projeto>_archive/ criado (context/plan/overdev/index/audit/scan/qa)
- [ ] <projeto>_auth_<lang> — IdP com baseline 2FA (ID≠email, senha+OTP, passkey, ReBAC, OIDC/JWKS)
- [ ] <projeto>_authfront — front de auth
- [ ] app principal delega ao auth por OIDC/PKCE (sem login embutido)
- [ ] <projeto>_<ctx>_<lang> — contextos do produto (segurança, testes/pentest, observabilidade, independência de runtime)
- [ ] <projeto>_front — front do produto (schematize-web)
- [ ] root de cada repo limpo + CLAUDE.md aplicado
- [ ] <projeto>_ops — control plane (promoção, seed, user+systemd por app, nproc, comandos idempotentes)
- [ ] CI com deploy gated (só promove verde; hml/prd pelo fluxo)
- [ ] DoD (§35) como gate de entrega em cada repo
- [ ] observabilidade integrada (health/logs/métricas/tracing) em todo serviço + ops
- [ ] efeito externo em sink de fábrica: .env.example com APP_ENV + MAIL_PROVIDER=sink + TEST_MAIL_DOMAIN + MAIL_MAX_PER_RUN, Mailpit no compose de dev, guard no provider com teste da recusa, fixtures só no domínio de teste, gate no CI
- [ ] domínio de teste test.<domain> em rota nula (null MX + SPF -all + DMARC p=reject) provado por dig (schematize-infra)
- [ ] índice/MAPA (§39) gerado e no archive
- [ ] overdev disponível; /scaffold-check passa com zero itens do piso faltando
```
