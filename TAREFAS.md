# Tarefas — HCLEAN

Estado em 20/08/2026. Ordenado por prioridade.

---

## Concluído

- **Pop-up de orçamento** substituiu a página de contato em todo o site. As
  quantidades são parametrizadas por produto (`data/quote.ts`) e chegam ao
  e-mail como linhas da tabela principal. Verificado de ponta a ponta com
  Playwright.
- **Página `/contato` removida**, com redirect 308 para a home — a URL estava
  indexada e circula em assinaturas.
- **Ornamentos de marca** em todas as páginas, via `<Ornament>`.
- **Auditoria zerada**: de 200 ocorrências para 2 (o 404 respondendo 404).
  Corrigidos hidratação quebrada, contraste da marca e alvos de toque no mobile.

---

## 1. Regras de quantidade — confirmar com a fábrica

Em `data/quote.ts`.

| Produto | Regra atual | Situação |
| --- | --- | --- |
| Barreiras SeaFence / ABFence | metro linear livre, mín. 1 m | decidido |
| Manta absorvente (3 linhas) | múltiplos de 200 un | decidido |
| Kit SOPEP | 50, 100, 200 e 1.000 L, múltipla escolha + qtd | decidido |
| Tanque Terrestre | C × L × A livres + qtd | decidido |
| Kit Primeiro Atendimento | unidade livre | ok |
| **Cordão, rolo, travesseiro, barreiras absorventes** | unidade livre | **falta o fardamento de cada** |
| **Turfa orgânica** | quilo, livre | **confirmar se é saco fechado** |

## 2. Vídeo do hero

- [ ] Colocar `hero.mp4` (e `hero.webm`, se houver) em `public/video/` e trocar
      `HERO_VIDEO` em `app/page.tsx` para
      `{ mp4: '/video/hero.mp4', webm: '/video/hero.webm' }`.
      Enquanto não existir, o hero usa o poster, que é foto real da operação.

## 3. Imagens

- [ ] **Barreira em tiras e barreira flocada dividem a mesma foto** — era a
      única disponível no site antigo, apesar de serem produtos distintos
      (12× vs 6× de absorção).
- [ ] Turfa orgânica: a foto é da embalagem, não do material.
- [ ] Só existe **uma** foto de operação real (`barreira-em-operacao.webp`), em
      1024×680 — fica mole em tela cheia. Vale conseguir os originais.
- [ ] Logos de clientes, órgãos e certificados: seção de maior peso comercial,
      ainda não existe.

## 4. Conteúdo pendente

- [ ] **Depoimentos**: o site antigo tinha a seção com texto de exemplo
      ("Insira aqui o depoimento do cliente"). Não migrei placeholder — se
      houver depoimentos reais, é a seção de maior peso comercial a acrescentar.
- [ ] **Kit SOPEP por capacidade**: o texto cita 50, 100, 200 e 1.000 L numa
      página só. Vale separar se cada capacidade tiver ficha própria.
- [ ] CNPJ e endereço no rodapé, se for para constar.

## 5. Produção

- [ ] **SMTP real.** Hoje o `.env` local aponta para um capturador em
      `127.0.0.1:2525`. Para valer:
      `SMTP_HOST=smtp.gmail.com`, `SMTP_PORT=587`,
      `SMTP_USER=contato.hcleanoil@gmail.com` e uma **Senha de App** em
      `SMTP_PASS` (myaccount.google.com/apppasswords — a senha da conta não
      funciona com 2FA).
- [ ] Definir onde hospedar e apontar `NEXT_PUBLIC_API_URL` e `CORS_ORIGINS`.

## 6. Dívidas técnicas

- [ ] O `hclean-designsystem.html` na raiz está com as edições da primeira
      abordagem (errada), não o original. Restaurar ou remover.
- [ ] `apps/frontend/fonts/static/` tem 18 `.ttf` de peso fixo que não são
      usados — o site carrega só o variável de `src/fonts/`. ~4 MB de peso morto.

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

A auditoria percorre as 13 rotas em desktop e mobile, checando erros de
console, requisições falhas, vazamento horizontal, imagens distorcidas ou sem
alt, hierarquia de títulos, contraste e alvos de toque. Salva um screenshot de
cada página em `scripts/screenshots/`.
