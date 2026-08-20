# Estudo — proposta em PDF automática após o formulário

**Pergunta:** o cliente preenche o formulário, recebe o e-mail padrão, e logo
em seguida recebe a proposta em PDF que o `RODAR.bat` gera hoje. É possível?

**Resposta curta:** sim, mas não com o gerador como ele está. Ele depende do
Microsoft Word instalado na máquina, e isso não sobrevive num servidor. Há três
caminhos; o recomendado está na seção 5.

---

## 1. O que o gerador faz hoje

`hclean-docs/HCLEAN/gerar_proposta.py` — 671 linhas, 80 propostas emitidas em
2026. Fluxo:

1. Lê e incrementa `contador_propostas.json` (hoje em `{"2026": 80}`).
2. Pergunta no terminal: empresa, estado, prazo.
3. Mostra os 20 produtos em 7 grupos e deixa marcar.
4. Para cada produto marcado, pergunta preço (sugerindo o padrão) e quantidade.
5. Decide o frete: **CIF** se for Sudeste e o total ≥ R$ 1.000; senão **FOB**.
6. Abre o `.docx` modelo, **remove as seções dos produtos não escolhidos**,
   preenche a tabela de preços e troca os marcadores.
7. Exporta o PDF.

O passo 6 é o mais valioso: a proposta sai com o catálogo enxuto, só do que o
cliente pediu. Não é um documento genérico.

### Como o PDF é gerado — o ponto crítico

```
atualizar_sumario_e_exportar_pdf_word()   →  Word via COM (PowerShell)
exportar_pdf_libreoffice()                →  fallback
```

O Word é necessário porque o modelo tem **sumário automático**, que só é
recalculado abrindo o documento num editor de verdade. Nesta máquina:

| | |
| --- | --- |
| Microsoft Word | instalado (16.0) |
| LibreOffice | **não instalado** |

Ou seja: hoje o PDF **só sai com o Word**, e o Word não roda em servidor Linux.
Automatizá-lo via COM num servidor Windows é possível, mas a Microsoft não
suporta esse uso — trava em diálogo modal, vaza processo e cai sob concorrência.

---

## 2. Preços: 17 de 20 produtos têm valor

**Com preço padrão** — proposta sai sem intervenção:

| Produto | Preço | Unidade |
| --- | --- | --- |
| Barreira Seafence | R$ 190,00 | metro |
| Kit SOPEP | R$ 300,00 | unidade (50 L) |
| Turfa orgânica | R$ 10,00 | kg |
| Barreira absorvente em tiras / flocada | R$ 21,00 | metro |
| Manta — branca / cinza | R$ 2,20 | pacote |
| Manta — verde | R$ 2,64 | pacote |
| Cordão — branca / cinza | R$ 9,00 | unidade |
| Cordão — verde | R$ 10,80 | unidade |
| Rolo — branca / cinza | R$ 540,00 | rolo |
| Rolo — verde | R$ 648,00 | rolo |
| Travesseiro — branca / cinza | R$ 6,00 | unidade |
| Travesseiro — verde | R$ 7,20 | unidade |

**Sem preço** — sempre manual, como você previu:

- Barreira **AB-Fence**
- **Kit Primeiro Atendimento**
- **Tanque Terrestre** 15 m³

Também vale notar: o Kit SOPEP tem preço só para 50 L. O comentário no código
registra 100 L = R$ 750 e 200 L = R$ 950, mas isso não está no dicionário — e o
site já pergunta a capacidade. Dá para cobrir os quatro sem trabalho extra.

---

## 3. Divergências entre o site e o gerador

Precisam ser resolvidas antes de qualquer automação, senão a proposta sai com
número errado.

| Item | Site pede | Gerador cobra | Risco |
| --- | --- | --- | --- |
| **Manta** | unidades, múltiplos de 200 | **por pacote**, R$ 2,20 | 400 unidades = 2 pacotes = R$ 4,40, **não** R$ 880 |
| **Rolo** | unidades | "rolo ou metro", R$ 540 | ambíguo: R$ 540 é o rolo inteiro ou o metro? |
| **Turfa** | unidades, mín. 10 | **por kg**, R$ 10,00 | quantos kg tem a unidade? |
| **Barreiras** | metro linear | metro | ok |
| **Cordão, travesseiro** | unidade | unidade | ok |

A da manta é a mais séria: erra o total em 200×. Nenhuma automação deve ser
ligada antes de alinhar essas quatro unidades.

---

## 4. O que já existe a favor

- O formulário **já coleta quantidade por variante** — foi exatamente o que
  construímos nesta semana. É o insumo que o gerador pede no passo 4.
- Os identificadores casam quase 1 para 1. `data/quote.ts` usa
  `branca-manta`, `sopep-200`; o Python usa `manta_lb`, `kit_sopep`. Um mapa de
  20 linhas resolve.
- O backend já envia dois e-mails e sabe montar HTML da marca. Anexar um PDF é
  uma linha no nodemailer.

---

## 5. Três caminhos

### A. Assistido — o gerador puxa o lead (recomendado para começar)

O formulário grava o pedido; o `RODAR.bat` passa a oferecer "usar um pedido
recebido" antes de perguntar tudo no terminal.

```
formulário → e-mail padrão + registro (como hoje)
           → pedido salvo em fila (JSON)
RODAR.bat  → lista pedidos pendentes → você escolhe
           → campos já preenchidos, você confere preço e envia
```

- **Prazo:** 1 a 2 dias.
- **Risco:** baixo. Nada muda no que já funciona; o gerador ganha uma entrada.
- **Ganho:** acaba a redigitação e o erro de transcrição. Você continua vendo
  cada proposta antes de sair — o que importa, já que 3 produtos não têm preço.
- **Limite:** não é automático. Depende de alguém rodar o `.bat`.

### B. Automático com Word numa máquina Windows

Um serviço na máquina que tem Word vigia a fila e gera o PDF sozinho.

- **Prazo:** 4 a 6 dias.
- **Risco:** alto. O Word via COM não é suportado para uso servidor: trava em
  diálogo, vaza processo, não aguenta concorrência. Exige a máquina ligada,
  logada, com watchdog. Um travamento silencioso significa lead sem resposta.
- **Ganho:** o cliente recebe a proposta em minutos.

### C. Automático com um renderizador de PDF próprio

Trocar o `.docx` + Word por um template HTML renderizado por navegador
(a mesma engine do Playwright que já está no projeto).

- **Prazo:** 6 a 10 dias.
- **Risco:** médio. Some a dependência do Word; roda em Linux, em container,
  sem licença. O custo é reconstruir o layout da proposta em HTML e validar
  contra as 80 já emitidas.
- **Ganho:** automação de verdade, e o mesmo template serve para o gerador
  manual. O sumário deixa de ser problema — em HTML ele é trivial.

---

## 6. Recomendação

**Fazer A agora, avaliar C depois; não fazer B.**

Motivos:

1. **Três produtos não têm preço.** Se o cliente pedir AB-Fence, Kit Primeiro
   Atendimento ou Tanque, nenhuma automação resolve — alguém precisa cotar.
   Um fluxo que às vezes manda proposta e às vezes não é pior, para o cliente,
   do que um que sempre manda em algumas horas.

2. **Proposta comercial com preço é documento que compromete.** Errar por causa
   da divergência de unidade da manta custa mais caro que a espera.

3. **O caminho B parece o atalho, mas é o mais frágil.** Word automatizado em
   servidor falha em silêncio, e o sintoma é justamente o lead não atendido.

4. **A resolve a maior parte da dor** — a redigitação — em 1 ou 2 dias, sem
   arriscar o que já funciona.

### Antes de qualquer opção

- [ ] Alinhar as unidades da tabela da seção 3, principalmente a manta.
- [ ] Confirmar os preços de Kit SOPEP 100 L, 200 L e 1.000 L.
- [ ] Decidir o que fazer quando o pedido inclui item sem preço: segurar a
      proposta inteira, ou mandar parcial avisando que o restante vem depois.

---

## 7. Sobre a página de produtos

Ponto que você levantou à parte: o site antigo mostra **Cordão Absorvente** e
**Rolo Absorvente** como produtos próprios em `/home/products/`, com card na
vitrine. Hoje eles existem no site novo só como formatos dentro das linhas
branca, cinza e verde — aparecem na página da linha, não na vitrine.

Os dois modelos se defendem:

- **Como está** reflete a fábrica: o mesmo cordão existe nas três linhas, e o
  que muda é o líquido que absorve.
- **Como era** ajuda quem procura "cordão absorvente" no Google e não sabe o
  que é "linha branca".

Dá para ter os dois: manter as linhas como estão e acrescentar páginas por
formato, que reúnem as três variantes lado a lado. São 6 URLs novas
(cordão, manta, rolo, travesseiro, barreira em tiras, barreira flocada) sem
duplicar conteúdo, porque cada uma compara as três linhas em vez de repetir.

Não fiz — é decisão de estrutura, não correção de bug. Diga se quer.
