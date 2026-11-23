<!-- Palette: #F4C7C3 rose · #E57373 coral · #C94A4A red · #B23A48 wine · #FAF3EB off-white -->

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&height=220&color=0:F4C7C3,100:C94A4A&text=AcessScan&fontColor=FAF3EB&fontSize=48&fontAlignY=40&desc=auditoria%20de%20acessibilidade%20%C2%B7%20projeto%20em%20constru%C3%A7%C3%A3o&descAlignY=60&descSize=16&descColor=FAF3EB" alt="header" width="100%" />

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=500&size=18&pause=1200&color=B23A48&center=true&vCenter=true&width=640&lines=WCAG+2.1+na+pr%C3%A1tica;PHP+%2B+Chromium+headless;Automatizar+o+que+d%C3%A1%2C+avisar+do+que+n%C3%A3o+d%C3%A1" alt="typing" />

<br/>

![Status](https://img.shields.io/badge/status-fase_1_·_runner_node-E57373?style=for-the-badge&labelColor=F5EDE3)
![Backlog](https://img.shields.io/badge/backlog-9_issues_abertas-B23A48?style=for-the-badge&labelColor=F5EDE3)
![License](https://img.shields.io/badge/license-MIT-C94A4A?style=for-the-badge&labelColor=F5EDE3)

![PHP](https://img.shields.io/badge/PHP_8.2+-C94A4A?style=for-the-badge&logo=php&logoColor=FAF3EB)
![SQLite](https://img.shields.io/badge/SQLite_WAL-E57373?style=for-the-badge&logo=sqlite&logoColor=FAF3EB)
![Node.js](https://img.shields.io/badge/Node.js-B23A48?style=for-the-badge&logo=node.js&logoColor=FAF3EB)
![axe-core](https://img.shields.io/badge/axe--core_4.14-C94A4A?style=for-the-badge&labelColor=F5EDE3)

</div>

---

> 🚧 **Projeto em construção.** O repositório foi criado, a arquitetura foi decidida em documento, e o código da primeira fase ainda está sendo escrito. Este README descreve **o que o AcessScan vai ser**, **as decisões arquiteturais já tomadas** e **o que vem primeiro**. Não é um README de "tudo funciona" — é um README de "isto é o que estamos construindo e por quê".

---

## 🌷 O que é o AcessScan

**AcessScan** é uma ferramenta de auditoria de acessibilidade web: você entrega uma URL, ela roda um conjunto de verificações WCAG 2.1 em um navegador headless real e devolve um relatório legível, com níveis de severidade, trecho de HTML ofensor, critério WCAG e — quando faz sentido — captura de tela do elemento.

A ideia não é despejar o JSON de uma engine de acessibilidade e chamar de relatório. É transformar o resultado bruto do axe-core em **leitura organizada por severidade**, com o critério WCAG traduzido (não a sigla `wcag111`, mas "1.1.1 Non-text Content"), com o valor do `alt` efetivamente extraído, e com o que **não** foi possível automatizar deixado explícito. Algo que sirva para **agir** em cima, não só para arquivar.

> *Automatizar o que dá, avisar do que não dá.*

---

## 🎯 O que ele vai verificar

O escopo inicial cobre cinco grupos de checagem:

| Verificação | Como |
|:--|:--|
| **Alt text** | Regras `image-alt`, `object-alt`, `role-img-alt`, `area-alt` do axe-core + **regra customizada `image-alt-value`** que extrai o valor de `alt` de cada imagem (ver [Decisões](#-decisões-arquiteturais)) |
| **Contraste** | `color-contrast` do axe-core + exibição da razão calculada a partir dos dados de cor retornados |
| **Ordem de headings** | `heading-order` do axe + passe customizado para níveis pulados e `h1` ausente/duplicado |
| **Rótulos de formulário** | `label`, `select-name`, `input-button-name`, `textarea-name`, etc. |
| **Focus trap** | Simulação de teclado (Tab) no navegador — **heurística**, rotulada como "possível trap" |

Cada achado carrega: regra, severidade (`error` / `warning` / `info`), critério WCAG, trecho de HTML, seletor CSS, nota de falha e — na Fase 2 — screenshot recortado do elemento.

---

## 🏗️ Arquitetura

Separação entre **orquestração (PHP)** e **execução no navegador (Node + Chromium headless)**. Checagens como ordem de headings e focus trap só fazem sentido em uma página renderizada com JavaScript ativo, e o ecossistema de automação de navegador é maduro em Node. O PHP fica com o que é seu: validação, persistência, correlação de resultados, classificação de severidade, relatório e API.

O Node é **deliberadamente burro**: recebe uma URL, abre o navegador, injeta o axe configurado, executa, e devolve o JSON bruto do axe. Não transforma, não classifica, não persiste.

```
┌──────────────┐     URL      ┌─────────────────────┐
│  PHP         │─────────────▶│  Node + Puppeteer    │
│  (orquestra) │              │  + axe-core          │
│              │◀─────────────│  (executa)           │
└──────────────┘   JSON bruto └─────────────────────┘
       │
       ▼
┌──────────────┐
│  SQLite (WAL)│
└──────────────┘
```

O contrato entre PHP e Node ainda está em aberto (fila, HTTP ou `proc_open`). A decisão será tomada quando o runner Node da Fase 1 estiver funcional — ver [Decisões abertas](#-decisões-abertas).

---

## 📐 Decisões arquiteturais

As decisões abaixo estão fechadas e documentadas em detalhe no repositório (pasta `docs/decisions/`). Cada uma lista **o que foi decidido**, **por quê** e **qual trade-off foi aceito**.

| # | Decisão | Resumo |
|:--|:--|:--|
| A1 | Separação PHP × Node | PHP orquestra; Node executa no navegador |
| A2 | axe-core como motor de regras | O axe responde "esta regra falhou"; transformar em relatório é responsabilidade do AcessScan |
| A3 | Contrato PHP ↔ Node | **Aberto** — decidir após Fase 1 |
| A4 | Modelo normalizado | `scans` + `rules` (versionada por `axe_version`) + `rule_tags` + `findings` com `custom_data` JSON |
| A5 | `display_severity` no PHP | Calculado on-the-fly a partir de `axe_result` + `custom_data`; não persistido |
| A6 | Custom check como **regra separada** | `image-alt-value` retorna `undefined` e anexa `data`; **não** é anexada a `image-alt` (evita regressão de veredito) |
| A7 | `alt=""` é **warning**, não erro | Imagem decorativa legítima pela WCAG; exibida como aviso pedagógico |
| A8 | Uma URL por execução | Simplifica schema e correlação; múltiplas URLs ficam para depois |
| A9 | SQLite em **WAL mode** | Leitores não bloqueiam escritor (e vice-versa) |
| A10 | Roadmap em fases | Fase 0 a 4; PHP só entra na Fase 3 |

### Modelo de dados (resumo do DER)

```
scans (id, url, timestamp, axe_version, user_agent, window_*, tool_options_json, status, error_message)
  │
  └─ findings (id, scan_id, rule_id, rule_version, target, html, failure_summary, axe_result, custom_data)
        │
        └─ rules (id, axe_version, impact, description, help, help_url)
              │
              └─ rule_tags (rule_id, rule_version, tag)
```

Duas colunas merecem destaque:

- **`axe_result`** guarda o que **o axe disse** (`violation`, `pass`, `incomplete`). É dado bruto, imutável.
- **`custom_data`** é JSON genérico, populado pelas regras customizadas. Hoje carrega `{alt, hasValue, isEmpty}` para a regra `image-alt-value`; amanhã carregará dados do focus trap.

A separação entre "o que o axe disse" e "o que o AcessScan exibe" é **princípio de projeto**, não detalhe de implementação. `display_severity` nunca é persistido — é derivado.

---

## 🗺️ Roadmap

O desenvolvimento é **de baixo para cima**, começando pelo runner Node e deixando o PHP (zona de conforto do autor) para o final. Sem ver o JSON do axe na mão, toda modelagem de banco seria chute.

| Fase | Escopo | Status |
|:--|:--|:--|
| **0** | Ponte conceitual: rodar o axe via CLI, inspecionar o JSON, entender `violations`/`passes`/`incomplete`/`inapplicable` | ✅ concluída |
| **1** | Runner Node mínimo: `image-alt` + `image-alt-value`, uma URL, sem PHP, sem persistência | 🚧 em andamento |
| **2** | As outras quatro verificações no runner: `color-contrast`, `heading-order` + passe customizado, rótulos de formulário, focus trap | ⏳ pendente |
| **3** | Orquestração PHP: contrato PHP↔Node, schema SQLite, persistência, classificação de severidade, relatório, API | ⏳ pendente |
| **4** | Refatoração: separação de camadas no runner, testes automatizados, CI | ⏳ pendente |

A **Fase 1** está fechada quando, rodando contra uma página local com três imagens (sem `alt`, com `alt=""`, com `alt` preenchido), o JSON devolvido tiver:

- `image-alt` → 1 violação (`#semAlt`) e 2 passes (`#altVazio`, `#AltEscrito`)
- `image-alt-value` → 3 nós em `incomplete`, cada um com `data = {alt, hasValue, isEmpty}` correto
- O processo termina com exit 0 e **não deixa Chrome órfão**

---

## 🔓 Decisões abertas

Documentadas para não serem esquecidas. Cada uma tem impacto arquitetural e será fechada na fase correspondente.

| # | Questão | Fase |
|:--|:--|:--|
| B1 | Contrato PHP ↔ Node: fila, HTTP ou `proc_open`? | 3 |
| B2 | Tabela explícita de `display_severity` (de-para testável) | 3 |
| B3 | Estratégia de screenshot recortado (`element.screenshot()` vs `clip`) | 2 |
| B4 | Formato do relatório (HTML, JSON, PDF?) | 3 |
| B5 | Implementação do focus trap (loop de Tab + detecção de ciclo) | 2 |
| B6 | Passe customizado de `heading-order` (níveis pulados, `h1` duplicado) | 2 |
| B7 | Tradução `wcag111` → "1.1.1 Non-text Content" (tabela estática?) | 3 |

---

## 🧭 Princípios de projeto

Três regras que orientam as decisões e que qualquer contribuição deve respeitar:

1. **Uma fonte de verdade por dado.** O veredito sobre falha WCAG é **sempre** do axe. O AcessScan não recalcula contraste, não reavalia `image-alt`, não reclassifica `violations`. A camada própria é **apresentação e enriquecimento** (extrair o valor do `alt`, reclassificar `pass` para warning), nunca re-veredito.

2. **Custom checks não alteram veredito de regras WCAG.** Toda check que apenas reporta dado vive em **regra separada**, com tags próprias, e retorna `undefined` (não `true`/`false`). Anexar uma check a uma regra WCAG faria o `any` da regra ter um `undefined`, migrando nós de `violations` para `incomplete` — regressão funcional.

3. **O runner é burro.** Node executa e devolve. PHP transforma, correlaciona, classifica e persiste. Se uma lógica pode morar no PHP, ela mora no PHP.

---

## 🛠️ Requisitos técnicos (resumo)

- **PHP** 8.2+ (Fase 3 em diante)
- **SQLite** com `PRAGMA journal_mode=WAL` (Fase 3)
- **Node.js** LTS (Fase 1)
- **axe-core** 4.14 (pinado; a versão no `package.json` deve casar com o `axe_version` no JSON)
- **Puppeteer** com Chromium embutido (não depende do Chrome do sistema)

Segurança mínima: URL só aceita `http:`/`https:`; se o transporte for `proc_open`, PHP **nunca** interpola a URL na string de comando (usar forma de array + `escapeshellarg`).

---

## 📚 Documentação do projeto

- `docs/decisions/` — uma ADR (Architecture Decision Record) por decisão da tabela acima
- `docs/specs/runner-node.md` — spec do runner Node (Fase 1)
- `docs/requirements.md` — requisitos funcionais, regras de negócio e não-funcionais
- `docs/der.md` — diagrama entidade-relacionamento completo, dicionário de dados

*(A pasta `docs/` será criada junto com o primeiro código da Fase 1.)*

---

## 📄 Licença

MIT. Ver `LICENSE`.

---

<div align="center">

*Construído com foco em aprender acessibilidade pela prática — e em não fingir que automação resolve tudo.*

</div>
