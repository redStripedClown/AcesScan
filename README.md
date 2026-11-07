<!-- Palette: #F4C7C3 rose · #E57373 coral · #C94A4A red · #B23A48 wine · #FAF3EB off-white -->

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&height=220&color=0:F4C7C3,100:C94A4A&text=AcessScan&fontColor=FAF3EB&fontSize=48&fontAlignY=40&desc=auditoria%20de%20acessibilidade%20%C2%B7%20projeto%20em%20constru%C3%A7%C3%A3o&descAlignY=60&descSize=16&descColor=FAF3EB" alt="header" width="100%" />

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=500&size=18&pause=1200&color=B23A48&center=true&vCenter=true&width=640&lines=WCAG+2.1+na+pr%C3%A1tica;PHP+%2B+Chromium+headless;Automatizar+o+que+d%C3%A1%2C+avisar+do+que+n%C3%A3o+d%C3%A1" alt="typing" />

<br/>

![Status](https://img.shields.io/badge/status-planejamento-E57373?style=for-the-badge&labelColor=F5EDE3)
![Backlog](https://img.shields.io/badge/backlog-9_issues_abertas-B23A48?style=for-the-badge&labelColor=F5EDE3)
![License](https://img.shields.io/badge/license-MIT-C94A4A?style=for-the-badge&labelColor=F5EDE3)

![PHP](https://img.shields.io/badge/PHP_8.2+-C94A4A?style=for-the-badge&logo=php&logoColor=FAF3EB)
![SQLite](https://img.shields.io/badge/SQLite-E57373?style=for-the-badge&logo=sqlite&logoColor=FAF3EB)
![Node.js](https://img.shields.io/badge/Node.js-B23A48?style=for-the-badge&logo=node.js&logoColor=FAF3EB)
![axe-core](https://img.shields.io/badge/axe--core-C94A4A?style=for-the-badge&labelColor=F5EDE3)

</div>

---

> 🚧 **Este projeto acabou de começar.** O repositório foi criado agora, o código ainda não existe e o escopo inicial está organizado como um backlog de issues. Este README descreve **o que o AcessScan vai ser**, **por que ele existe** e **o que vem primeiro**. Conforme as issues forem fechando, esta página será atualizada.

---

## 🌷 O que é o AcessScan

**AcessScan** vai ser uma ferramenta de auditoria de acessibilidade web: você entrega uma URL, ela roda um conjunto de verificações WCAG 2.1 em um navegador headless real e devolve um relatório legível com níveis de severidade e capturas de tela.

A ideia não é despejar JSON de uma engine de acessibilidade e chamar de relatório. É transformar os achados em uma leitura organizada — por severidade, com o trecho de HTML ofensor, o critério WCAG correspondente e, quando fizer sentido, a razão de contraste calculada. Algo que sirva para **agir** em cima, não só para arquivar.

> *Automatizar o que dá, avisar do que não dá.*

---

## 🎯 O que ele vai verificar

O escopo inicial cobre cinco grupos de checagem, escolhidos por serem os que mais aparecem em auditorias reais e os que a automação consegue tratar com alguma confiança:

| Verificação | Como |
|:--|:--|
| **Alt text ausente** | Regras `image-alt`, `object-alt`, `role-img-alt`, `area-alt` do axe-core |
| **Contraste** | `color-contrast` do axe-core + recálculo da razão em PHP para exibir o valor exato |
| **Ordem de headings** | `heading-order` do axe + passe customizado para detectar níveis pulados e h1 ausente/duplicado |
| **Rótulos de formulário** | `label`, `select-name`, `input-button-name`, `textarea-name`, etc. |
| **Focus trap** | Simulação de teclado (Tab) no navegador — **heurística**, rotulada como "possível trap" |

Cada violação carrega: regra, severidade, critério WCAG, trecho de HTML ofensor, seletor CSS, nota de falha e — quando possível — um screenshot recortado do elemento.

---

## 🏗️ Arquitetura planejada

A separação entre **orquestração (PHP)** e **execução no navegador (Node + Chromium)**. Checagens como ordem de headings e focus trap só fazem sentido em uma página renderizada com JavaScript ativo, e o ecossistema de automação de navegador é maduro em Node. O PHP fica com o que é seu: validação, fila, persistência, relatório e API.
