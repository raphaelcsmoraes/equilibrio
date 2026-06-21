# Guia de Desenvolvimento — Equilíbrio

Este documento descreve a arquitetura técnica do Equilíbrio e como modificar o jogo.

---

## Arquitetura

O Equilíbrio é um **arquivo único auto-contido** (`index.html`). HTML, CSS e JavaScript vivem juntos no mesmo arquivo, sem dependências externas. Funciona offline e em qualquer navegador moderno.

```
index.html
├── <head>
│   ├── <meta> — configurações, SEO, favicon inline
│   └── <style> — todo o CSS
├── <body>
│   ├── #game-screen — tela principal do jogo
│   │   ├── .header-row — título, timer, botões
│   │   ├── #rules-panel — painel de regras (preenchido pelo JS)
│   │   ├── .legend — legenda dos símbolos
│   │   ├── #board-wrap — grade 7x7 (6 células + indicadores)
│   │   ├── .controls — botões Dica / Verificar / Reiniciar
│   │   └── #status — mensagens de feedback
│   └── #completed-screen — tela de conclusão
└── <script>
    ├── i18n (dicionário PT/EN)
    ├── Fuso Brasília
    ├── localStorage
    ├── Gerador de puzzles (backtracking)
    ├── Cronômetro
    ├── Ícones SVG
    ├── Renderização + indicadores em tempo real
    ├── Lógica do jogo (clique, verificar, dica, reset)
    └── Inicialização
```

---

## O Gerador de Puzzles

Esta é a parte mais sofisticada do código. Ao contrário de bancos de puzzles fixos, o Equilíbrio **gera um tabuleiro novo todo dia** via algoritmo determinístico.

### Por que backtracking?

No Reflexo, cada linha do tabuleiro é **independente** — as regras só atuam dentro de cada linha, então é possível enumerar todas as linhas válidas e combiná-las. No Equilíbrio, as restrições das colunas **acoplam** as linhas entre si: a escolha de uma célula afeta tanto a contagem da linha quanto a da coluna. Isso torna a enumeração direta impraticável — backtracking é a abordagem certa.

### `generateGrid(rng)`

Preenche a grade célula por célula (esquerda → direita, cima → baixo) usando backtracking com RNG:

```javascript
function generateGrid(rng){
  // grid 6x6 iniciado com null
  // colCts: contagem de símbolos por coluna

  function fill(pos){
    if(pos === 36) return true; // todas as células preenchidas

    var r = Math.floor(pos / 6);
    var c = pos % 6;

    // Símbolos válidos: não excedeu 2 na linha NEM na coluna
    var valid = symbols.filter(function(s){
      return rowCt[s] < 2 && colCts[c][s] < 2;
    });

    // Tenta cada símbolo válido em ordem aleatória (via RNG)
    for cada símbolo em valid (embaralhado):
      grid[r][c] = símbolo
      colCts[c][símbolo]++
      if(fill(pos + 1)) return true  // achou solução
      colCts[c][símbolo]--           // backtrack
      grid[r][c] = null

    return false; // não há solução por este caminho
  }
}
```

**Performance:** para uma grade 6×6 com essas restrições, o backtracking converge muito rapidamente. Em testes, 30 dias de puzzles são gerados em menos de 100ms no total.

### `countSolutions(partial, max)`

Conta quantas soluções existem para um tabuleiro parcialmente preenchido. Usado para garantir que cada puzzle tenha **exatamente uma resposta**. Interrompe ao encontrar `max` soluções (geralmente `max=2` — se há mais de 1, precisamos de mais pistas).

### `pickGivens(grid, rng, extra)`

Seleciona quais células serão reveladas como pistas:

1. Embaralha todas as 36 posições
2. Adiciona posições uma a uma ao conjunto de pistas
3. Após cada adição, verifica se `countSolutions == 1`
4. Para quando a solução se tornar única
5. Adiciona `extra` células a mais (para ajustar dificuldade)

```
extra alto = mais pistas = mais fácil
extra baixo = menos pistas = mais difícil
```

### Semente Diária

```javascript
var seed = ((dayNumber * 2654435761) ^ 0x9E3779B9) >>> 0;
```

`dayNumber` é o número de dias desde o epoch no fuso de Brasília. Como a semente é sempre a mesma para uma data, **todos os jogadores veem o mesmo puzzle** e o tabuleiro não muda ao recarregar a página.

---

## Indicadores em Tempo Real

A grande diferença visual do Equilíbrio em relação ao Reflexo.

### Layout do tabuleiro

O `#board-wrap` usa CSS Grid com **7 colunas × 7 linhas**:

```
[0,0] [0,1] [0,2] [0,3] [0,4] [0,5] [row-ind-0]
[1,0] [1,1] [1,2] [1,3] [1,4] [1,5] [row-ind-1]
[2,0] [2,1] [2,2] [2,3] [2,4] [2,5] [row-ind-2]
[3,0] [3,1] [3,2] [3,3] [3,4] [3,5] [row-ind-3]
[4,0] [4,1] [4,2] [4,3] [4,4] [4,5] [row-ind-4]
[5,0] [5,1] [5,2] [5,3] [5,4] [5,5] [row-ind-5]
[col-ind-0] [col-ind-1] ... [col-ind-5] [corner]
```

### `isRowOk(r)` e `isColOk(c)`

Verificam se uma linha/coluna está completamente preenchida com exatamente 2 de cada símbolo.

### `updateIndicators()`

Chamada após cada clique do jogador. Percorre as 6 linhas e 6 colunas, atualizando a classe e o texto de cada indicador:

```javascript
// Indicador verde com ✓ quando satisfeito
ind.className = 'row-ind ok';
ind.textContent = '✓';

// Indicador neutro quando incompleto
ind.className = 'row-ind';
ind.textContent = '·';
```

---

## Sistema de Idiomas (i18n)

Dicionário completo PT/EN no topo do script. Detecção automática pelo idioma do navegador, com preferência salva em `localStorage('equilibrio_lang')`.

```javascript
function detectLang(){
  var saved = localStorage.getItem('equilibrio_lang');
  if(saved === 'pt' || saved === 'en') return saved; // preferência salva vence
  var nav = (navigator.language || 'en').toLowerCase();
  return nav.indexOf('pt') === 0 ? 'pt' : 'en';
}
```

A função `applyLanguage()` atualiza todos os textos estáticos da interface. O painel de regras é construído dinamicamente por `buildRulesPanel()`, que injeta os ícones SVG reais dos símbolos no texto.

---

## 💾 LocalStorage Schema

```javascript
// Dados do jogador
'equilibrio_player' → { lastPlayedKey: "2026-06-20", streak: 5 }

// Estado do jogo de hoje
'equilibrio_game_2026-06-20' → { completed: true, time: 287 }

// Horário de início do cronômetro (para sobreviver a refreshes)
'equilibrio_timer_start_2026-06-20' → 1718870400000

// Preferência de idioma
'equilibrio_lang' → "pt" | "en"
```

### Depuração no Console

```javascript
// Ver dados do jogador
JSON.parse(localStorage.getItem('equilibrio_player'))

// Ver estado do jogo de hoje
JSON.parse(localStorage.getItem('equilibrio_game_' + '2026-06-20'))

// Simular novo dia (apaga estado de hoje)
localStorage.removeItem('equilibrio_game_2026-06-20')
localStorage.removeItem('equilibrio_timer_start_2026-06-20')

// Resetar tudo
localStorage.clear()
```

---

## ⏰ Fuso Horário (Brasília)

O puzzle muda à meia-noite no horário de Brasília (UTC-3), independente de onde o jogador esteja.

```javascript
function getBrasiliaNow(){
  var now = new Date();
  return new Date(now.getTime() + (now.getTimezoneOffset() * 60000) - 10800000);
}
```

---

## 🎮 Fluxo do Jogo

### Abertura
```
Abrir página
  ↓
Já completou hoje? → Sim → Tela de conclusão (tempo + streak + contagem)
  ↓ Não
Gera grade com backtracking (semente = dia atual)
  ↓
Seleciona pistas garantindo solução única
  ↓
Inicia cronômetro (ou retoma do localStorage)
  ↓
Renderiza tabuleiro + indicadores
```

### Jogando
```
Clique em célula → handleClick(r, c)
  ↓
Cicla: vazio → círculo → triângulo → quadrado → vazio
  ↓
Chama updateIndicators() — acende indicadores em tempo real
  ↓
(Opcional) useHint() → revela célula aleatória + cooldown 15s
  ↓
Clica "Verificar" → verify()
  ↓
Verifica linhas E colunas
  ↓
Erros? → Destaca linhas com fundo vermelho, mostra mensagem
  ↓ Nenhum erro
Para cronômetro → atualiza streak → salva → showCompletedScreen()
```

---

## 🔧 Como Modificar

### Ajustar Dificuldade

```javascript
var difficultyByWeekday = {
  1: {extra: 8}, // Segunda - mais fácil
  2: {extra: 6},
  3: {extra: 5},
  4: {extra: 4},
  5: {extra: 3},
  6: {extra: 2},
  0: {extra: 1}  // Domingo - mais difícil
};
```

Aumente `extra` para revelar mais células (mais fácil). Diminua para revelar menos (mais difícil).

### Mudar Cores

```css
:root {
  --accent: #2D6A4F;    /* Cor principal (verde) */
  --circle: #1A7F5A;    /* Cor do círculo */
  --triangle: #C0392B;  /* Cor do triângulo */
  --square: #6C3483;    /* Cor do quadrado */
}
```

### Adicionar um Quarto Símbolo

Para mudar de 3 para 4 símbolos (e a grade de 6×6 para 8×8):
1. Adicione o novo símbolo em `var symbols = [...]`
2. Adicione o ícone SVG em `var ICONS = {...}`
3. Adicione a cor em `:root` e `colorMap`
4. Ajuste o gerador para verificar `< 2` → `< (gridSize/numSymbols)`
5. Atualize o CSS Grid para `repeat(8, 1fr) 22px`

---

**Versão:** 1.0.0
