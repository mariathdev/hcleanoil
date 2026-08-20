# Tabela de preços — HCLEAN

Transcrição da tabela enviada pelo cliente em 20/08/2026, com a comparação
contra `PRECOS_PADRAO` em `hclean-docs/HCLEAN/gerar_proposta.py`.

**O site não exibe preço** — ele coleta quantidade e a proposta é montada pelo
gerador. Este documento existe para alinhar as duas pontas antes de qualquer
automação (ver [ESTUDO-PROPOSTA-AUTOMATICA.md](ESTUDO-PROPOSTA-AUTOMATICA.md)).

## O que ainda falta decidir

Cinco pontos. Nada disso foi alterado no código — preço errado numa proposta
comercial custa caro, então nenhum valor foi inventado.

| # | Pendência | Por quê |
| --- | --- | --- |
| 1 | **Manta**: 3 espessuras, 3 preços | o gerador guarda um só |
| 2 | **Travesseiro**: 4 medidas | nenhuma bate com as 2 que o site anuncia |
| 3 | **Barreira absorvente**: unidade × metro | site pede unidade, gerador cobra metro — erra 3× |
| 4 | **Manta**: unidade × pacote | site pede unidade, gerador cobra pacote — erra 200× |
| 5 | **Kit SOPEP 1.000 L** | o site oferece, a tabela não tem preço |

Já resolvido nesta rodada: AB-Fence (R$ 230/m), Kit Primeiro Atendimento
(R$ 950/un), turfa (1 kg = 1 unidade) e o tanque seguindo manual com alerta.

---

## Regra das linhas

| Linha | Preço |
| --- | --- |
| Branca | valor de tabela |
| **Cinza** | **igual à branca** |
| **Verde** | **branca + 20%** |

Já implementada no gerador. Os rolos existem nas três linhas — confirmado pelo
cliente e já refletido no site.

## Tabela enviada

| Item | Medida | Preço | Unidade |
| --- | --- | --- | --- |
| Cordão absorvente branco | 0,76 cm × 1,20 m | R$ 9,00 | un |
| Manta absorvente branca | 0,4 × 0,5 × 0,03 m | R$ 2,40 | un |
| Manta absorvente branca | 0,4 × 0,5 × 0,02 m | R$ 2,20 | un |
| Rolo absorvente branco | 0,76 cm × 1,20 m | R$ 540,00 | un |
| Manta absorvente branca | 0,4 × 0,5 × 0,04 m | R$ 2,20 | un |
| Travesseiro absorvente | 0,23 × 0,23 × 0,015 | R$ 4,00 | un |
| Travesseiro absorvente | 0,45 × 0,45 × 0,015 | R$ 6,00 | un |
| Travesseiro absorvente | 0,45 × 0,45 × 0,022 | R$ 13,00 | un |
| Travesseiro absorvente | 0,45 × 0,45 × 0,023 | R$ 14,00 | un |
| Barreira absorvente | — | R$ 63,00 | un (R$ 21,00 o metro) |
| Tanque | — | fazer cotação | — |
| Kit SOPEP 50 L | 50 L | R$ 300,00 | un |
| Kit SOPEP 100 L | 100 L | R$ 750,00 | un |
| Kit SOPEP 200 L | 200 L | R$ 950,00 | un |
| Barreira de contenção Fortflex 550 mm | — | R$ 190,00 | metro linear |

---

## Divergências com o gerador — confirmar antes de automatizar

O gerador guarda **um** preço por produto; a tabela lista **várias medidas** do
mesmo item, com preços diferentes. Onde há mais de uma, o gerador ficou com uma
delas e as outras se perderam.

### 1. Manta: três espessuras, um preço no gerador

| Espessura | Tabela | Gerador |
| --- | --- | --- |
| 0,03 m | R$ 2,40 | — |
| 0,02 m | R$ 2,20 | **R$ 2,20** |
| 0,04 m | R$ 2,20 | — |

O gerador usa 2,20. Falta saber qual é a espessura padrão de venda e se as
outras devem virar itens separados.

### 2. Travesseiro: quatro medidas, uma no gerador

| Medida | Tabela | Gerador |
| --- | --- | --- |
| 0,23 × 0,23 × 0,015 | R$ 4,00 | — |
| 0,45 × 0,45 × 0,015 | R$ 6,00 | **R$ 6,00** |
| 0,45 × 0,45 × 0,022 | R$ 13,00 | — |
| 0,45 × 0,45 × 0,023 | R$ 14,00 | — |

O site anuncia **dois** tamanhos (0,23 × 0,23 × 0,05 e 0,45 × 0,45 × 0,05),
que não batem com nenhuma espessura da tabela. Precisa alinhar.

### 3. Barreira absorvente: unidade e metro

A tabela traz **R$ 63,00 a unidade** e **R$ 21,00 o metro** — a unidade tem
3 m, o que fecha. O gerador só tem os R$ 21,00 por metro; se o pedido vier em
unidades, o cálculo sai errado por 3×.

O formulário do site pede **unidade** para as barreiras absorventes.

### 4. Kit SOPEP: só o 50 L está no dicionário

100 L e 200 L existem apenas como comentário no código, então caem para
digitação manual. **Decidido:** cadastrar os três (50, 100 e 200 L).

O site também oferece **1.000 L**, que não está na tabela — falta o preço, ou
tirar a opção do formulário.

### 5. Turfa orgânica — resolvido

**1 kg = 1 unidade**, mínimo de 10. A unidade do site e a do gerador passam a
significar a mesma coisa; o preço de R$ 10,00 vale por unidade.

### 6. Itens sem preço — resolvido, menos o tanque

| Item | Decisão |
| --- | --- |
| **Barreira AB-Fence** | **R$ 230,00 o metro linear** |
| **Kit Primeiro Atendimento** | **R$ 950,00 a unidade** |
| **Tanque Terrestre** | segue **manual** — a tabela diz "fazer cotação" |

Para o tanque, o e-mail da proposta deve trazer no topo, em vermelho:
*"essa cotação exige personalização"*.

Com isso, **só o tanque** impede a proposta de sair pronta — os outros dois
saem do balcão de exceções.

---

## Divergências de unidade entre site e gerador

Repetido do estudo, porque é o que mais afeta o valor da proposta:

| Item | Site pede | Gerador cobra | Situação |
| --- | --- | --- | --- |
| **Manta** | unidades, múltiplos de 200 | **por pacote** | **erra 200×** se 1 pacote = 200 un |
| **Barreira absorvente** | unidades | por metro | **erra 3×** (a unidade tem 3 m) |
| **Rolo** | unidades | "rolo ou metro" | R$ 540 é o rolo inteiro ou o metro? |
| Turfa | unidades, mín. 10 | por kg | resolvido: 1 kg = 1 unidade |
| Barreiras de contenção | metro | metro | ok |
| Cordão, travesseiro | unidade | unidade | ok |

Restam **três**: manta, barreira absorvente e rolo.
