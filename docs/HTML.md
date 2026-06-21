# Documentação: HTML (Estrutura)

Este documento descreve a **estrutura HTML** do Equilíbrio. Aqui não há lógica nem estilo, apenas a organização dos elementos na tela.

> **Nota:** O Equilíbrio é um projeto de arquivo único. Todo o HTML descrito aqui vive dentro de `index.html`, junto com o CSS (no `<head>`) e o JavaScript (antes do `</body>`). Esta documentação separa as camadas apenas para fins didáticos.

---

## Visão Geral

A página tem **duas telas** que nunca aparecem ao mesmo tempo:

1. **Tela do jogo** (`#game-screen`) — onde o jogador resolve o puzzle
2. **Tela de conclusão** (`#completed-screen`) — aparece quando o puzzle é concluído

O JavaScript alterna entre elas via `style.display`.

---

## O `<head>`

Configurações invisíveis ao usuário:

- `<meta charset="UTF-8">` — permite acentos e caracteres especiais
- `<meta name="viewport">` — responsividade em celulares
- `<title>` — texto da aba do navegador (bilíngue)
- `<meta name="description">` — usado pelo Google e ao compartilhar links
- `<meta name="theme-color">` — cor da barra do navegador em celulares (verde)
- `<link rel="icon">` — favicon inline como SVG com os três símbolos do jogo
- `<style>` — todo o CSS (documentado em `CSS.md`)

---

## A Tela do Jogo (`#game-screen`)

### Cabeçalho (`.header-row`)

Faixa no topo do card:

- `.avatar` — círculo verde com ícone de balança (símbolo do Equilíbrio)
- `.game-name` — texto "Equilíbrio"
- `#day-indicator` — dia da semana ("Equilíbrio de Quarta")
- `#timer` — cronômetro com ícone de relógio
- `#lang-btn` — botão PT/EN para troca de idioma
- `#rules-btn` — botão "i" que abre o painel de regras

### Painel de Regras (`#rules-panel`)

Começa vazio no HTML — o JavaScript o preenche via `buildRulesPanel()` no idioma correto, com os ícones reais dos símbolos. Inicia escondido (`display:none`) e aparece ao clicar no botão "i".

### Legenda (`.legend`)

Explica os três símbolos:
- ○ Círculo (verde)
- △ Triângulo (vermelho)
- □ Quadrado (roxo)

Cada item tem o SVG do símbolo + um `<span>` com ID (`#legend-circle`, `#legend-triangle`, `#legend-square`) que o JavaScript preenche com o texto traduzido.

### Tabuleiro (`#board-wrap`)

A `<div>` principal do jogo, vazia no HTML. O JavaScript cria dinamicamente:

- **36 células** (`.game-cell`) — os quadradinhos do puzzle
- **6 indicadores de linha** (`.row-ind`) — um à direita de cada linha
- **6 indicadores de coluna** (`.col-ind`) — um abaixo de cada coluna
- **1 canto vazio** (`.corner`) — célula (6,6) sem função visual

Total: **49 elementos** num CSS Grid 7×7.

Diferentemente do Reflexo (que tinha 36 células num Grid 6×6), o Equilíbrio usa 7 colunas e 7 linhas para acomodar os indicadores sem sair do fluxo do layout.

### Botões de Controle (`.controls`)

- `#hint-btn` — Dica (com `<span id="hint-text">` que vira contagem regressiva)
- `#verify-btn` — Verificar (com `<span id="verify-text">`)
- `#reset-btn` — Reiniciar (com `<span id="reset-text">`)

Os textos nos `<span>` são separados do ícone SVG para facilitar a tradução.

### Mensagem de Status (`#status`)

Linha de texto abaixo dos botões para mensagens do jogo: erros, dicas reveladas, células vazias restantes.

---

## A Tela de Conclusão (`#completed-screen`)

Aparece quando o puzzle é resolvido. Contém:

- `#completed-title` — "Puzzle completo!" / "Puzzle complete!"
- `#completion-time` — tempo que levou (ex: "Tempo: 04:32")
- `#streak-number` — número grande com os dias consecutivos
- `#streak-label` — "Dias consecutivos 🔥"
- `#next-puzzle-label` — "Próximo puzzle em:"
- `#countdown` — contagem regressiva até a meia-noite de Brasília

---

## Acessibilidade

- `<h1 class="sr-only">` — título oculto visualmente mas lido por leitores de tela
- `aria-label` — em células e botões (ex: "linha 3, coluna 2: vazia")
- `role="button"` — informa que células são clicáveis
- `tabIndex` — permite navegar com a tecla Tab
- `aria-hidden="true"` — nos SVGs decorativos, para não poluir leitores de tela

---

## Diferença Estrutural em Relação ao Reflexo

| | Reflexo | Equilíbrio |
|---|---|---|
| **Grid do tabuleiro** | 6×6 (36 elementos) | 7×7 (49 elementos) |
| **Linha do espelho** | Sim (borda tracejada na col 3) | Não |
| **Indicadores** | Nenhum | 12 (6 linhas + 6 colunas) |
| **Cracks (raios)** | Sim | Não |
