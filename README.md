# Avaliação Funnelsheet - Ítalo Iung

## Tema
Decisão técnica (Next.js vs Nuxt 3) + estimativas — Funnelsheet (Home + Blog/Prismic

## Contexto
```
Queremos lançar o site da Funnelsheet com SEO excelente, schema/metadados completos, sitemap robusto e CRO alto. Conteúdo diário do Blog via Prismic; páginas institucionais seguindo FSD + Atomic; estilo Swiss Minimalist. Deploy preferencial em Vercel com ISR.
```

## Objetivo desta avaliação (timebox 2h):
1. Escolher Next.js ou Nuxt 3 (decisão binária).
2. Explicar 3–5 razões objetivas (SEO/SSR-SSG-ISR, schema/meta/i18n,
sitemap/indexação, Core Web Vitals, DX/manutenção/roadmap, custos/time-to-market).
3. Desenhar plano técnico para Home e Blog integrados ao Prismic.
4. Estimativas (horas, riscos, premissas) usando o template.

## Escopo do MVP (apenas análise/planejamento, sem código obrigatório):
- Homepage: hero, prova social, benefícios, CTAs/utilitários.
- Blog: /blog (listagem; paginação futura) e /blog/[slug] (TOC, SEO/Schema Article, 404).
- Prismic: autores, tipos de posts; SliceZone/slices; cache/normalização via proxy /api/cms.
- SEO/CRO: canonical, breadcrumbs, schema (Article/BlogPosting, BreadcrumbList, Organization), sitemap.xml, robots.txt, noindex em utilitários; eventos (CTAs, scroll, WhatsApp), pixels e Consent Mode v2.

## Padrões e premissas:
- FSD (app/pages/widgets/features/entities/shared) + Atomic em shared/ui.
- Acessibilidade AA; TypeScript; testes básicos na pipeline; logs de build.
- Sem i18n nesta fase (arquitetura preparada).
- Riscos a observar: CSP (nonces), hydration mismatch em slices dinâmicos, schema duplicado, Consent Mode v2 mal configurado

---

## Stack escolhida (tecnologias utilizadas)
Além do framework responsável pela renderização de UI e da API, a arquitetura será composta por um conjunto de tecnologias-chave que suportam SEO, escalabilidade e experiência de desenvolvimento.

### Framework Fullstack (UI & API)
Dada a preferência por **deploy em ambiente Vercel** e da exigência do cache *ISR* consolidado, a escolha mais apropriada para a base do projeto é o **Next.js**.

No entanto, é válido salientar que o Nuxt também pode ser hospedado na Vercel — apesar de ter [ambiente próprio para deploy em nuvem](https://nuxt.studio/) —, fornece nativamente cache *ISR* ao mesmo tempo que favorece o desenvolvimento de MVPs sem comprometer a escalabilidade através de sintaxe flexível, pouco verbosa, e com ênfase em *DX*.

#### Razões elencadas
- **Next.js + Vercel**: integração nativa com ISR/SSG/SSR, cache incremental, observabilidade e suporte oficial a Preview Mode.
- **SEO consolidado**: bibliotecas e patterns maduros para schema/meta, sitemap, robots.txt e Core Web Vitals.
- **Ecosistema React**: a natureza pouco opinativa da biblioteca demanda arquitetura robusta, enquanto a ampla adoção da tecnologia oferece vasto repositório de bibliotecas para CRO, tracking e otimização de desempenho, reduzindo *time-to-market*.

### Biblioteca UI + framework CSS
- **Shadcn/UI**: base para **Atomic Design**, compondo átomos, moléculas e organismos, unindo:
    - **Radix UI**: garante **acessibilidade** e padrões **WAI-ARIA** para componentes base.
    - **TailwindCSS**: framework CSS utilitário que facilita consistência visual e estilo Swiss Minimalist.

### Demais tecnologias
- **Prismic (CMS headless)**: posts, autores e slices de conteúdo editorial.
- **TypeScript**: tipagem forte e segurança em todas as camadas.
- **Playwright**: testes E2E para smoke.
- **React Testing Library + Jest**: testes unitários.
- **DOMPurify**: sanitização de HTML no proxy `/api/cms`.
- **Sentry**: monitoramento de erros em produção.
- **Lighthouse CI**: validação de SEO/Core Web Vitals prévia ao deploy.
- **Consent Mode v2 + GTM**: controle de pixels e eventos condicionado a consentimento

---

## Plano técnico resumido
### Arquitetura de alto nível
- **Monorepo (pnpm/workspaces) com pacotes**: `app/` (Next), `ui/` (shared/ui - atomic design), `lib/cms/` (proxy utils + normalizers), `config/` (schema, SEO helpers), `tests/`.
- **FSD + Atomic**:
    - **Folders**: `app/pages`, `app/widgets`, `app/features`, `app/entities`, `shared/`. `shared/ui` organizado em `atoms/`, `molecules/`, `organisms/` com barrels (`index.ts`) por nível.
- **Data flow**:
    - Front-end consome composables/hooks (ex.: useHomeData(), useAllPosts(), usePostBySlug()).
    - Hooks chamam o proxy serverless /api/cms/* (no Next API Routes / Edge Functions) que fala com Prismic, normaliza e aplica cache/headers (stale-while-revalidate).
- **Cache & ISR**:
    - **Home**: pré-rendered (SSG) + ISR revalidate n minutos (ex.: 300s) — alta performance.
    - **Listing /blog**: SSG/ISR com reval n min (configurável).
    - **Post** `/blog/[slug]`: SSG com reval janela configurável por ambiente (ex.: 15–60 min) + Preview mode para editores.
- **Proxy** `/api/cms`: rota de API serverless para consumir Prismic, aplicar cache (*stale-while-revalidate*), sanitizar HTML e normalizar JSON.

### Slices e UI
- **SliceZone**: integra slices editoriais do Prismic, isolados da lógica de negócio.
- **UI primitives**: em `shared/ui` (Atomic Design).
- **Fallback**: slice desconhecida renderiza `UnknownSlice` para evitar *build breaks*.

### ISR/Revalidação
- **Homepage**: SSG + revalidate a cada 5 min.
- **Blog listing**: ISR a cada 10 min.
- **Posts individuais**: janela configurável (15–60 min).
- **Preview Mode**: atualização instantânea para autores.

### SEO & CRO
- **Meta Helpers & Schema centralizado**: utilidades `getMetaForRoute()` para renderizar os metadados inseridos via Prismic e `buildSchema()` gera JSON-LD para Organization, BreadcrumbList, BlogPosting.
- **Sitemap e robots**: rotas /sitemap.xml e /robots.txt dinâmicas.
- **TOC local + CTAs persistentes**: aumenta CRO e UX.
- **Eventos CRO**: clicks em CTAs, scroll depth, WhatsApp, diagnósticos, todos versionados em dataLayer.

---

## Estimativas
### Arquitetura & Setup
- Monorepo/Repo, TypeScript, lint/test: 6h
- SEO/meta/sitemap/robots + schema base: 8h
- Ponte CMS (proxy `/api/cms`, cache, normalização): 10h

### Homepage
- Layout + seções (hero, prova social, benefícios, CTAs): 14h
- Eventos CRO (CTAs, scroll, WhatsApp/diagnóstico): 6h

### Blog
- Listagem /blog (ordenar, paginação futura, tags): 8h
- Post /blog/[slug] (TOC, SEO/Schema Article, 404): 10h
- Tipos Prismic (autor, post) + SliceZone: 10h

### Infra/DevOps
- Deploy Vercel + ISR + observabilidade: 4h

### QA/Performance
- Lighthouse/Core Web Vitals, schema validator, e2e smoke: 8h

Total estimado: 84h
Custo (hora €Z): € (84 × Z)
Prazo (calendário): 29/09/2025 ~ 17/10/2025

---

## Riscos e mitigações
- **CSP/nonces**: risco de bloquear scripts ~ configurar middleware + rotinas de testes em ambiente de homologação.
- **Hydration mismatch em slices**: mitigado com renderização de placeholders e normalização/sanitização de dados durante deploy e revalidates.
- **Schema duplicado**: centralizar **buildSchema()** e validação prévia deploy através de CI/CD.
- **Consent Mode v2 mal configurado**: testar cenários de consentimento/negativa em E2E.
