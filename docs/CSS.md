# Documentação: CSS (Estilo)

Este documento descreve o **CSS** do Equilíbrio — cores, tamanhos, layout e estados visuais. O CSS vive dentro do bloco `<style>` no `<head>` do `index.html`.

---

## Paleta de Cores (`:root`)

O Equilíbrio usa uma paleta **verde-esmeralda** como identidade visual, diferenciando-se claramente do azul do Reflexo.

```css
:root {
  --bg: #EEF2EE;           /* Fundo da página (verde bem claro) */
  --card-bg: #FFFFFF;      /* Fundo do card (branco) */
  --border: #D8E4D8;       /* Bordas suaves (esverdeadas) */
  --border-strong: #B8CCB8;/* Bordas mais marcadas */
  --text-primary: #1A2E1A; /* Texto principal (verde bem escuro) */
  --text-secondary: #5A7A5A;/* Texto secundário */
  --bg-secondary: #F0F5F0; /* Fundo secundário (células fixas, hover) */
  --accent: #2D6A4F;       /* Cor principal (verde-esmeralda) */

  /* Cores dos símbolos */
  --circle: #1A7F5A;       /* Círculo — verde */
  --triangle: #C0392B;     /* Triângulo — vermelho */
  --square: #6C3483;       /* Quadrado — roxo */

  /* Feedback */
  --danger: #B23A33;       /* Erro (vermelho) */
  --success: #2D6A4F;      /* Sucesso (verde — mesmo que --accent) */
  --warning: #A04000;      /* Aviso (laranja escuro) */

  --radius-md: 8px;        /* Cantos médios */
  --radius-lg: 14px;       /* Cantos grandes (card) */
}
```

Para mudar o tema do jogo inteiro, altere `--accent` e as cores dos símbolos aqui.

---

## O Card (`.card`)

Retângulo branco que contém o jogo:

- `max-width: 400px` — não estica demais em telas largas
- `border-radius: var(--radius-lg)` — cantos arredondados
- Centralizado pelo `body` com `display: flex; justify-content: center`

---

## O Tabuleiro (`#board-wrap`)

A principal diferença estrutural em relação ao Reflexo. Usa **CSS Grid 7×7**:

```css
#board-wrap {
  display: grid;
  grid-template-columns: repeat(6, 1fr) 22px; /* 6 colunas iguais + 1 para indicadores */
  grid-template-rows: repeat(6, 1fr) 22px;    /* 6 linhas iguais + 1 para indicadores */
  gap: 4px;
  max-width: 295px;
}
```

- As 6 primeiras colunas e linhas são as células do jogo
- A 7ª coluna contém os **indicadores de linha** (`.row-ind`)
- A 7ª linha contém os **indicadores de coluna** (`.col-ind`)
- O elemento (7,7) é um `.corner` vazio

**Por que 295px?** Para que as células tenham ~40px cada (295 ÷ 6 ≈ 49px, menos gap), ficando quadradas e confortáveis para toque em celular.

---

## As Células (`.game-cell`)

```css
.game-cell {
  aspect-ratio: 1/1;        /* Sempre quadrada */
  display: flex;            /* Centraliza o ícone */
  align-items: center;
  justify-content: center;
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  cursor: pointer;
  min-height: 36px;
}
```

- `aspect-ratio: 1/1` garante que as células sejam sempre quadradas, em qualquer tamanho de tela
- `min-height: 36px` garante tamanho mínimo confortável em celulares

### Estados das Células

| Classe | Quando aparece | O que faz |
|--------|---------------|-----------|
| `.fixed` | Célula já preenchida (pista) | Fundo cinza, cursor não-clicável |
| `.error-row` | Linha com erro ao verificar | Fundo levemente vermelho |
| `:hover` | Mouse sobre célula | Fundo cinza claro |
| `:focus` | Foco por teclado | Contorno verde (--accent) |

**Nota importante:** `.error-row` só muda o `background-color`, nunca `border` ou `padding`. Isso garante que as células nunca mudem de tamanho ao mostrar erros — um problema clássico de layout que causaria desalinhamento no grid.

---

## Os Indicadores (`.row-ind`, `.col-ind`)

```css
.row-ind, .col-ind {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 700;
  color: var(--text-secondary);
  border-radius: 4px;
  transition: all .2s;
}

/* Estado satisfeito */
.row-ind.ok, .col-ind.ok {
  color: var(--success); /* verde */
  font-size: 13px;
}
```

Quando uma linha ou coluna está corretamente preenchida, a classe `.ok` é adicionada e o indicador:
- Muda de cor cinza para verde
- Aumenta levemente de tamanho (11px → 13px)
- Mostra "✓" em vez de "·"

A transição de 0.2s cria uma animação suave ao acender.

---

## Os Botões

Mesmo padrão do Reflexo — borda fina, fundo branco, com estados `:hover`, `:active` e `:disabled`.

```css
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

O estado `:disabled` é usado no botão de dica durante o cooldown de 15 segundos.

---

## A Tela de Conclusão (`.completed-screen`)

- `display: none` por padrão
- `.big-number` — o número de streak em 56px, verde-esmeralda, peso 800
- `.countdown-time` — fonte monoespaçada via `font-variant-numeric: tabular-nums` (os números não "dançam" enquanto o tempo muda)

---

## Responsividade

O layout adapta a diferentes telas sem precisar de media queries específicas:

- `max-width` no card e no tabuleiro limitam o crescimento em telas grandes
- `aspect-ratio: 1/1` nas células mantém proporções em qualquer tamanho
- `1fr` nas colunas do grid divide o espaço disponível igualmente
- `padding: 16px` no `body` garante margens em celulares

---

## Diferenças Visuais em Relação ao Reflexo

| | Reflexo | Equilíbrio |
|---|---|---|
| **Cor principal** | Azul (#1B5FAD) | Verde-esmeralda (#2D6A4F) |
| **Fundo da página** | Bege (#F6F3EE) | Verde muito claro (#EEF2EE) |
| **Texto principal** | Quase preto (#231F1A) | Verde escuro (#1A2E1A) |
| **Grid do tabuleiro** | 6×6 | 7×7 (com indicadores) |
| **Linha do espelho** | Tracejada azul | Não existe |
| **Indicadores** | Não existe | 12 pontos/checkmarks |
