# Tarefas — HCLEAN

Estado em 20/08/2026.

---

## Concluído

- **Pop-up de orçamento** substituiu a página de contato em todo o site, com as
  quantidades parametrizadas por produto. Verificado no navegador.
- **Mínimos de pedido** confirmados com a fábrica e aplicados (ver tabela
  abaixo). `scripts/check-minimos.mjs` confere se o campo bate com a regra.
- **Ornamentos de marca** em todas as páginas.
- **Fontes** convertidas para WOFF2 e itálica removida: 591 kB → 103 kB.
- **SEO**: imagem de compartilhamento gerada no build, títulos e descrições
  dentro do limite do Google.
- **Vídeo do hero descartado** por decisão do cliente. O hero usa a foto da
  operação; o componente não tem mais o caminho de vídeo.
- **Depoimentos: nada a migrar.** O site antigo só tinha o texto padrão do
  Elementor ("Insira aqui o depoimento do cliente") com avatar genérico —
  publicar isso seria pior do que não ter a seção.
- **Auditorias**: página, pop-up, performance e SEO, todas passando.

### Mínimos de pedido (em `data/quote.ts`)

| Produto / formato | Unidade | Mínimo | Incremento |
| --- | --- | --- | --- |
| Barreira SeaFence / ABFence | metro linear | 1 | 1 |
| Cordão absorvente | unidade | 10 | 1 |
| Manta absorvente | unidade | 200 | 200 (pacote) |
| Barreira absorvente em tiras | unidade | 10 | 1 |
| Barreira absorvente flocada | unidade | 10 | 1 |
| Rolo absorvente | unidade | 1 | 1 |
| Travesseiro absorvente | unidade | 10 | 1 |
| Turfa orgânica | unidade | 10 | 1 |
| Kit SOPEP (50/100/200/1.000 L) | unidade | 1 | 1 |
| Kit Primeiro Atendimento | unidade | 1 | 1 |
| Tanque Terrestre | C × L × A + unidade | 1 | 1 |

---

## 1. Produção

- [ ] **SMTP.** Os campos estão em `apps/backend/.env.example`, com comentário
      em cada um. Copie para `.env` e preencha `SMTP_PASS` com uma **Senha de
      App** do Gmail (myaccount.google.com/apppasswords — a senha da conta não
      funciona com 2FA). As outras 10 variáveis já vêm com o valor de produção.
- [ ] Definir hospedagem e ajustar `NEXT_PUBLIC_API_URL` (frontend) e
      `CORS_ORIGINS` (backend) para o domínio real.

## 2. Imagens

- [ ] **Barreira em tiras e barreira flocada dividem a mesma foto** — era a
      única disponível no site antigo, apesar de serem produtos distintos
      (12× vs 6× de absorção).
- [ ] Turfa orgânica: a foto é da embalagem, não do material.
- [ ] Só existe **uma** foto de operação real, em 1024×680. Ela é o fundo do
      hero, ampliada; com o original em resolução maior, ganharia nitidez.
- [ ] Logos de clientes, órgãos e certificados — não existem no projeto.

## 3. Conteúdo

- [ ] **Kit SOPEP por capacidade**: o texto cita 50, 100, 200 e 1.000 L numa
      página só. Vale separar se cada capacidade tiver ficha própria.
- [ ] CNPJ e endereço no rodapé, se for para constar.
- [ ] Depoimentos reais, se houver — é a seção de maior peso comercial que o
      site não tem.

## 4. Dívidas técnicas

- [ ] O `hclean-designsystem.html` na raiz está com as edições da primeira
      abordagem (errada), não o original. Restaurar ou remover.
- [ ] `apps/frontend/fonts/` guarda os TTF originais (~4 MB) que não são
      usados — o site carrega os WOFF2 de `src/fonts/`. Mantidos como fonte
      para regerar; remover se não for preciso.

---

## Como rodar

```bash
# backend  (http://localhost:4000)
cd apps/backend && npm install && cp .env.example .env
npm run dev

# frontend (http://localhost:3000)
cd apps/frontend && npm install && npm run dev
```

### Testar o formulário sem SMTP real

```bash
cd apps/backend && npm run maildev     # SMTP falso na 2525 + caixa na 8025
```

Aponte o `.env` para ele (`SMTP_HOST=127.0.0.1`, `SMTP_PORT=2525`,
`SMTP_USER=dev`, `SMTP_PASS=dev`) e as mensagens aparecem em
http://localhost:8025, sem sair para a internet.

### Auditorias

```bash
cd apps/frontend
node scripts/audit.mjs                       # páginas: console, layout, contraste, toque
node scripts/audit-modal.mjs                 # pop-up: teclado, foco, validação, mobile
node scripts/check-minimos.mjs               # mínimos de pedido por formato

npm run build && npx next start -p 3100      # as duas abaixo precisam do build
node scripts/audit-perf.mjs http://localhost:3100   # peso, LCP, CLS
node scripts/audit-seo.mjs  http://localhost:3100   # metadados, JSON-LD, sitemap
```

### Regerar as fontes

Só é necessário se trocar o arquivo em `fonts/`:

```bash
cd apps/frontend && node scripts/fonts-to-woff2.mjs
```
