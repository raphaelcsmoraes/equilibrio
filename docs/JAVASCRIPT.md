# Documentação: JavaScript (Lógica)

Este documento descreve o **JavaScript** do Equilíbrio. O código vive dentro do bloco `<script>` no final do `index.html`, envolvido por uma IIFE `(function(){ ... })()` para isolar o escopo.

---

## 1. Sistema de Idiomas (i18n)

Dicionário completo com todos os textos do jogo em português e inglês.

| Função | O que faz |
|--------|-----------|
| `detectLang()` | Lê a preferência salva ou detecta o idioma do navegador |
| `buildRulesPanel()` | Constrói o painel de regras com os ícones reais dos símbolos |
| `applyLanguage()` | Aplica o idioma atual em toda a interface |

### Detecção de idioma

```javascript
function detectLang(){
  var saved = localStorage.getItem('equilibrio_lang');
  if(saved === 'pt' || saved === 'en') return saved; // preferência salva vence sempre

  var nav = (navigator.language || 'en').toLowerCase();
  return nav.indexOf('pt') === 0 ? 'pt' : 'en'; // pt-BR, pt-PT → pt; qualquer outro → en
}
```

A preferência salva tem prioridade sobre o idioma do navegador. Se o usuário clicar no botão PT/EN, a escolha é salva em `localStorage('equilibrio_lang')`.

---

## 2. Fuso Horário (Brasília)

O puzzle muda à meia-noite **no horário de Brasília** (UTC-3), independente de onde o jogador esteja.

| Função | O que faz |
|--------|-----------|
| `getBrasiliaNow()` | Data/hora atual convertida para UTC-3 |
| `getTodayKey()` | Data de hoje como `"YYYY-MM-DD"` (chave do dia) |
| `getWeekday()` | Dia da semana em Brasília (0=Dom, 6=Sáb) |
| `getSecondsUntilMidnight()` | Segundos até a próxima meia-noite (para o contador) |
| `formatTimeRemaining(s)` | Formata segundos em `HH:MM:SS` |

---

## 3. Persistência (localStorage)

| Função | O que faz |
|--------|-----------|
| `loadPlayerData()` | Lê streak e último dia jogado |
| `savePlayerData(data)` | Salva os dados do jogador |
| `updateStreak()` | Atualiza a sequência: +1 se jogou ontem, volta a 1 se pulou |

### Chaves utilizadas

```javascript
'equilibrio_player'                   // dados do jogador
'equilibrio_game_YYYY-MM-DD'          // se completou hoje e em quanto tempo
'equilibrio_timer_start_YYYY-MM-DD'   // timestamp de início do cronômetro
'equilibrio_lang'                     // preferência de idioma (pt ou en)
```

Todas as chaves incluem `equilibrio_` no prefixo para não colidir com as chaves do Reflexo (`reflexo_`) caso o jogador acesse os dois jogos no mesmo dispositivo.

---

## 4. Gerador de Puzzles

Esta é a parte mais sofisticada do Equilíbrio, e é onde ele difere fundamentalmente do Reflexo.

### Por que backtracking (e não enumeração)?

No **Reflexo**, cada linha é independente — as regras (simetria e contagem) só atuam dentro da linha. Isso permite enumerar todas as linhas válidas possíveis e escolher uma por uma.

No **Equilíbrio**, as restrições das colunas acoplam todas as linhas entre si. A escolha de um símbolo na célula (linha 1, coluna 3) afeta o que pode ir nas células (linha 2, coluna 3), (linha 3, coluna 3), etc. Enumeração linha por linha não funciona — backtracking é necessário.

### `makeRng(seed)` e `shuffleArr(arr, rng)`

Gerador de números pseudoaleatórios com semente (mulberry32). A mesma semente sempre gera a mesma sequência — isso garante que todos os jogadores vejam o mesmo puzzle no mesmo dia.

### `generateGrid(rng)` — O coração do gerador

Preenche a grade 6×6 célula por célula, da esquerda para a direita, de cima para baixo:

```
Para cada posição (0 a 35):
  r = posição ÷ 6   (linha)
  c = posição % 6   (coluna)

  símbolos válidos = aqueles que:
    - não apareceram 2+ vezes na linha atual
    - não apareceram 2+ vezes na coluna atual

  Tenta cada símbolo válido (em ordem aleatória via RNG):
    → preenche a célula
    → continua para a próxima posição (recursão)
    → se não houver solução: desfaz (backtrack) e tenta o próximo
```

**Performance:** para uma grade 6×6 com essas restrições, o backtracking converge em menos de 1ms por grade na maioria dos casos.

### `countSolutions(partial, max)` — Garantia de unicidade

Conta quantas soluções existem para um tabuleiro parcialmente preenchido. Usa o mesmo algoritmo de backtracking do `generateGrid`, mas com células fixas (as pistas) e contando soluções em vez de procurar uma.

O parâmetro `max` serve para interromper cedo: se `max=2` e já encontrou 2 soluções, para imediatamente. Se o resultado for `1`, o puzzle tem solução única.

### `pickGivens(grid, rng, extra)` — Seleção de pistas

1. Embaralha todas as 36 posições (via RNG)
2. Parte de uma grade vazia
3. Adiciona posições uma a uma
4. Após cada adição, chama `countSolutions(partial, 2)`
5. Para quando `countSolutions == 1` (solução única atingida)
6. Adiciona `extra` células a mais (para facilitar conforme o dia da semana)

```
extra = 8 → Segunda (muito fácil)
extra = 1 → Domingo (muito difícil)
```

### Semente diária

```javascript
var dayNumber = Math.floor(brasiliaNow / 86400000);
var seed = ((dayNumber * 2654435761) ^ 0x9E3779B9) >>> 0;
```

- `dayNumber` muda a cada dia no fuso de Brasília
- A semente é determinística: mesma data = mesmo puzzle = mesmos jogadores veem o mesmo
- Dois RNGs são criados: `rng` para o grid, `rng2` para as pistas (separados para reprodutibilidade)

---

## 5. Cronômetro

| Função | O que faz |
|--------|-----------|
| `getTimerKey()` | Chave do localStorage com a data de hoje |
| `getTimerStart()` | Lê ou cria o timestamp de início do dia |
| `formatTime(s)` | Formata em `MM:SS` |
| `updateTimerDisplay()` | Atualiza o texto na tela |
| `startTimer()` | Inicia o intervalo de contagem |
| `stopTimer()` | Para o intervalo |

### Por que o tempo sobrevive a refreshes

O cronômetro não conta "tique a tique". Ele guarda o **momento em que o jogador abriu o puzzle** (`getTimerStart`) e recalcula o tempo decorrido a cada segundo:

```javascript
elapsedSeconds = Math.floor((Date.now() - startTime) / 1000);
```

Ao recarregar a página, o `startTime` é recuperado do localStorage e o cálculo continua de onde parou. O botão "Reiniciar" não chama `startTimer()` — o tabuleiro é limpo mas o tempo segue.

---

## 6. Ícones SVG

```javascript
var ICONS = {
  circle:   '<circle cx="12" cy="12" r="8"/>',
  triangle: '<polygon points="12 4 21 20 3 20"/>',
  square:   '<rect x="3" y="3" width="18" height="18" rx="2"/>',
  clock:    '...',
  info:     '...'
}
```

`iconSvg(name, size, colorVar)` gera o SVG completo. Formas (circle, triangle, square) são `fill="currentColor"` (preenchidas). Ícones de interface (clock, info) são `stroke` (contorno).

---

## 7. Renderização e Indicadores

| Função | O que faz |
|--------|-----------|
| `isRowOk(r)` | Verifica se a linha r está completa e com 2-2-2 |
| `isColOk(c)` | Verifica se a coluna c está completa e com 2-2-2 |
| `updateIndicators()` | Atualiza todos os 12 indicadores (6 linhas + 6 colunas) |
| `render()` | Reconstrói toda a grade (células + indicadores) |

### Como os indicadores funcionam

Após cada clique, `render()` é chamado, que por sua vez chama `updateIndicators()`:

```javascript
function updateIndicators(){
  for(var r=0; r<6; r++){
    var ok = isRowOk(r);
    ind.className = 'row-ind' + (ok ? ' ok' : '');
    ind.textContent = ok ? '✓' : '·';
  }
  // mesmo para colunas
}
```

A transição CSS de 0.2s cria a animação suave ao acender verde.

---

## 8. Lógica do Jogo

| Função | O que faz |
|--------|-----------|
| `handleClick(r, c)` | Cicla o símbolo da célula: vazio → círculo → triângulo → quadrado → vazio |
| `verify()` | Verifica todas as linhas e colunas, marca erros |
| `resetBoard()` | Limpa células não-fixas, mantém cronômetro |
| `getRandomEmpty()` | Escolhe célula vazia aleatória (para dicas) |
| `useHint()` | Revela célula com o símbolo correto + cooldown 15s |
| `updateHintButton()` | Controla estado e texto do botão de dica |

### Como `verify()` funciona

Diferentemente do Reflexo (que só verificava linhas), o Equilíbrio verifica **linhas E colunas**:

```javascript
// Verifica cada linha
for(var r=0; r<6; r++){
  if(linha r está completa mas não tem 2-2-2) → marca como erro
}

// Verifica cada coluna
for(var c=0; c<6; c++){
  if(coluna c está completa mas não tem 2-2-2) → marca TODAS as linhas como erro
}
```

Quando uma coluna tem erro, todas as 6 linhas ficam marcadas (já que o erro de coluna afeta múltiplas linhas).

---

## 9. Carregamento Inicial

### `loadPuzzle()`

Junta tudo:
1. Calcula a semente do dia
2. Gera a grade com `generateGrid(rng)`
3. Seleciona pistas com `pickGivens(grid, rng2, config.extra)`
4. Popula `solution`, `fixedSet` e `board`
5. Chama `applyLanguage()`, `startTimer()` e `render()`

### Sequência de inicialização

```javascript
applyLanguage();    // aplica idioma detectado

if(!checkAndShowCompletedIfNeeded()){
  // se já completou hoje → mostra tela de conclusão
  loadPuzzle();     // senão → carrega o puzzle
}
```

---

## Resumo do Fluxo

```
Abrir página
  ↓
Já completou hoje? → Sim → Tela de conclusão
  ↓ Não
loadPuzzle() → gera grade + seleciona pistas
  ↓
render() → cria 36 células + 12 indicadores
  ↓
Jogador clica → handleClick() → render() → updateIndicators()
  ↓
(Opcional) useHint() → revela célula + cooldown 15s
  ↓
Clica "Verificar" → verify()
  ↓
Erros? → Destaca linhas + mensagem
  ↓ Nenhum erro
stopTimer() → updateStreak() → salva → showCompletedScreen()
```
