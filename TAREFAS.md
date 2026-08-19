# Tarefas — HCLEAN

Estado em 19/08/2026. Ordenado por prioridade.

---

## 1. Terminar a migração para o pop-up de orçamento

O pop-up está **construído e funcionando**, com as quantidades por produto já
parametrizadas. Falta trocar os CTAs que ainda navegam para `/contato`.

**Pronto:**
- `components/quote/QuoteModal.tsx` — pop-up em `<dialog>` nativo (foco preso,
  Esc, backdrop), com os campos do print: nome, e-mail, DDD+telefone, produto,
  quantidade e observações.
- `components/quote/QuoteProvider.tsx` — no layout; qualquer CTA abre o mesmo
  pop-up, com o produto pré-selecionado.
- `components/quote/QuoteButton.tsx` — botão que dispara o pop-up.
- `data/quote.ts` — regras de quantidade por produto (ver seção 2).
- Backend aceita e formata as quantidades no e-mail (`extractItems`).
- Cabeçalho já usa o pop-up.

**Falta:**
- [ ] Trocar `ButtonLink href="/contato"` por `<QuoteButton>` em:
  - `app/page.tsx` (hero e CTA final)
  - `app/produtos/page.tsx` e `app/produtos/[slug]/page.tsx`
  - `app/sobre/page.tsx`
  - `app/not-found.tsx`
  - `components/sections/Shared.tsx` (o `CTABanner` recebe `href`; trocar por
    callback ou por um `QuoteCTABanner`)
  - `components/site/Footer.tsx` (duas entradas na coluna Atendimento)
- [ ] Nos cartões e páginas de produto, passar `productSlug` para o pop-up já
      abrir com o item certo.
- [ ] **Remover a página `/contato`** (decisão do cliente: só pop-up).
      Ao remover, tirar também de `data/site.ts` (nav), `app/sitemap.ts` e
      `components/site/Footer.tsx`. Considerar um redirect de `/contato` para
      `/` em `next.config.mjs`, já que a URL estava indexada.
- [ ] Depois disso, apagar `components/sections/ContactForm.tsx` e o
      `app/contato/`, que ficam órfãos.

## 2. Regras de quantidade — confirmar com a fábrica

Em `data/quote.ts`. O que está valendo hoje:

| Produto | Regra atual | Confirmar |
| --- | --- | --- |
| Barreiras SeaFence / ABFence | metro linear livre, mín. 1 m | ok (decidido) |
| Manta absorvente (3 linhas) | múltiplos de 200 un | ok (decidido) |
| Cordão, rolo, travesseiro, barreiras absorventes | unidade livre | **falta o mínimo/fardamento de cada** |
| Turfa orgânica | quilo, livre | confirmar se é saco fechado |
| Kit SOPEP | 50, 100, 200 e 1.000 L, múltipla escolha + qtd | ok |
| Kit Primeiro Atendimento | unidade livre | ok |
| Tanque Terrestre | C × L × A livres + qtd | ok (decidido) |

## 3. Assets e ornamentos nas demais páginas

A home recebeu quatro ornamentos de marca (anel, ondas, anéis concêntricos,
malha de pontos), com os lados alternados para não empilharem.

- [ ] Aplicar o mesmo tratamento em **Produtos**, **Quem somos** e nas páginas
      de produto. Os componentes já existem em `components/sections/Shared.tsx`:
      `BrandWaveArt`, `BrandLinesArt`, `BrandRingsArt`, `BrandDotsArt`.
- [ ] As páginas de produto ainda são muito "texto em bloco" — merecem a mesma
      composição da home.

## 4. Vídeo do hero

- [ ] Colocar `hero.mp4` (e `hero.webm`, se houver) em `public/video/` e trocar
      `HERO_VIDEO` em `app/page.tsx` para
      `{ mp4: '/video/hero.mp4', webm: '/video/hero.webm' }`.
      Enquanto não existir, o hero usa o poster, que é foto real da operação.

## 5. Imagens

- [ ] **Barreira em tiras e barreira flocada dividem a mesma foto** — era a
      única disponível no site antigo, apesar de serem produtos distintos
      (12× vs 6× de absorção).
- [ ] Turfa orgânica: a foto é da embalagem, não do material.
- [ ] Só existe **uma** foto de operação real (`barreira-em-operacao.webp`), e
      ela é de 1024×680 — fica mole em tela cheia. Vale conseguir originais.
- [ ] Logos de clientes, órgãos e certificados: seção de maior peso comercial,
      ainda não existe.

## 6. Conteúdo pendente

- [ ] **Kit SOPEP por capacidade**: o texto cita 50, 100, 200 e 1.000 L numa
      página só. Vale separar se cada capacidade tiver ficha própria.
- [ ] **Depoimentos**: o site antigo tinha a seção com texto de exemplo
      ("Insira aqui o depoimento do cliente"). Não migrei placeholder — se
      houver depoimentos reais, é a seção de maior peso comercial a acrescentar.
- [ ] CNPJ e endereço no rodapé, se for para constar.

## 7. Dívidas técnicas

- [ ] O `hclean-designsystem.html` na raiz está com as edições da primeira
      abordagem (errada), não o original. O original está fora do repositório;
      restaurar ou remover o arquivo.
- [ ] `apps/frontend/fonts/static/` tem 18 `.ttf` de peso fixo que não são
      usados — o site carrega só o variável de `src/fonts/`. ~4 MB de peso morto.
- [ ] Rodar `node scripts/audit.mjs` depois das mudanças acima. Restam
      pendências de **alvo de toque no mobile** (links do menu com 26px de
      altura; o mínimo recomendado é 44px).

---

## Como rodar

```bash
# backend  (http://localhost:4000)
cd apps/backend && npm install && cp .env.example .env   # preencher SMTP_PASS
npm run dev

# frontend (http://localhost:3000)
cd apps/frontend && npm install && npm run dev

# auditoria visual e de acessibilidade, todas as rotas
cd apps/frontend && node scripts/audit.mjs
```

Para testar o formulário sem SMTP real, existe um capturador local de e-mails
que sobe em `http://localhost:8025` — ver histórico da conversa.
