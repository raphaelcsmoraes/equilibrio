# ⚖️ Equilíbrio

**Equilíbrio** é um quebra-cabeça diário de lógica bidimensional. Preencha uma grade 6×6 com círculos, triângulos e quadrados de forma que cada linha **e** cada coluna tenha exatamente 2 de cada símbolo.

🎮 **Jogue agora:** https://jogo-equilibrio.netlify.app

---

## Como Jogar

### O Tabuleiro

- Grade **6×6** com três símbolos: ○ Círculo, △ Triângulo, □ Quadrado
- Algumas células já estão preenchidas (pistas fixas)
- Preencha as células vazias clicando nelas para alternar entre os símbolos

### As Regras

1. **Cada linha** deve ter exatamente 2 círculos, 2 triângulos e 2 quadrados
2. **Cada coluna** deve ter exatamente 2 círculos, 2 triângulos e 2 quadrados

### Feedback em Tempo Real

- Os indicadores **ao lado de cada linha** (→) e **abaixo de cada coluna** (↓) ficam **verdes** com ✓ quando aquela linha ou coluna estiver correta
- Você não precisa esperar o "Verificar" para saber como está indo

### Como Ganhar

1. Preencha todas as células vazias
2. Garanta que todos os indicadores estejam verdes
3. Clique em "Verificar" — o puzzle estará completo!

---

## Características

- **Um puzzle novo todo dia** — gerado automaticamente à meia-noite (horário de Brasília)
- **Feedback em tempo real** — indicadores de linha e coluna que acendem ao serem completados
- **Dificuldade progressiva** — sobe suavemente de segunda (mais fácil) a domingo (mais difícil)
- **Sistema de dicas** — revela uma célula com cooldown de 15 segundos
- **Cronômetro persistente** — continua mesmo se a página for atualizada
- **Rastreamento de sequência** — dias consecutivos jogados
- **Tela de conclusão** — mostra tempo, streak e contagem regressiva para o próximo puzzle
- **Bilíngue PT/EN** — detecta o idioma do navegador automaticamente com botão de troca manual
- **Zero dependências** — funciona offline, sem instalar nada

---

## Diferenças em Relação ao Reflexo

| | Reflexo | Equilíbrio |
|---|---|---|
| **Símbolos** | Sol, Lua, Estrela | Círculo, Triângulo, Quadrado |
| **Restrição** | Por linha (horizontal) | Por linha E coluna (2D) |
| **Regra especial** | Pares "raio" (diferente) | Nenhuma |
| **Feedback** | Só ao verificar | Em tempo real |
| **Dificuldade** | Mais acessível | Mais desafiador |

O Equilíbrio é naturalmente mais difícil que o Reflexo porque cada célula afeta dois lugares ao mesmo tempo — a linha e a coluna.

---

## Como Instalar e Jogar Localmente

1. Baixe o arquivo `index.html`
2. Abra no seu navegador
3. Pronto — não precisa de servidor, instalação ou conexão com internet

---

## Como Publicar

### Netlify (recomendado)
1. Acesse [netlify.com](https://netlify.com)
2. Crie um novo projeto
3. Arraste o `index.html` na área de deploy
4. Pronto — URL gerada automaticamente

### GitHub Pages
1. Crie um repositório
2. Suba o `index.html`
3. Ative GitHub Pages nas configurações

---

## Tecnologias

- **HTML5** — estrutura e semântica
- **CSS3** — layout com Grid, variáveis de cor, responsividade
- **JavaScript (Vanilla)** — lógica, gerador de puzzles, persistência

Sem frameworks, sem bibliotecas, sem dependências externas.

---

## 📊 Sistema de Progresso

### Cronômetro Persistente
O tempo começa quando você abre o puzzle do dia. Se atualizar a página, o cronômetro continua de onde parou. O botão "Reiniciar" limpa o tabuleiro mas mantém o tempo.

### Streak (Sequência)
- Complete o puzzle do dia: +1 dia
- Pule um dia: contador volta a 1
- Salvo localmente no navegador

### Dificuldade por Dia da Semana
```javascript
var difficultyByWeekday = {
  1: {extra: 8}, // Segunda - mais fácil
  2: {extra: 6}, // Terça
  3: {extra: 5}, // Quarta
  4: {extra: 4}, // Quinta
  5: {extra: 3}, // Sexta
  6: {extra: 2}, // Sábado
  0: {extra: 1}  // Domingo - mais difícil
};
```
`extra` = células reveladas além do mínimo necessário para solução única.

---

## 💾 Armazenamento de Dados

Tudo salvo localmente no navegador (`localStorage`). Nenhum dado vai para servidores.

```javascript
// Dados do jogador
localStorage.getItem('equilibrio_player')
// { lastPlayedKey: "2026-06-20", streak: 5 }

// Estado do jogo de hoje
localStorage.getItem('equilibrio_game_2026-06-20')
// { completed: true, time: 287 }

// Horário de início do cronômetro
localStorage.getItem('equilibrio_timer_start_2026-06-20')
// 1718870400000 (timestamp em ms)

// Preferência de idioma
localStorage.getItem('equilibrio_lang')
// "pt" ou "en"
```

---

## 📁 Estrutura do Projeto

```
equilibrio/
├── index.html          # O jogo completo (HTML + CSS + JS num só arquivo)
├── README.md           # Este arquivo
├── DEVELOPMENT.md      # Guia técnico e como modificar
├── ROADMAP.md          # Planos futuros
├── CHANGELOG.md        # Histórico de versões
├── CONTRIBUTING.md     # Como contribuir
├── CODE_OF_CONDUCT.md  # Código de conduta
├── LICENSE             # Licença MIT
├── package.json        # Metadados
└── docs/
    ├── README.md       # Índice da documentação técnica
    ├── HTML.md         # Estrutura da página
    ├── CSS.md          # Estilos e layout
    └── JAVASCRIPT.md   # Lógica, gerador e mecânicas
```

---

## 🗺️ Roadmap

- [ ] Limite de dicas por dia
- [ ] Estatísticas detalhadas
- [ ] Temas customizáveis
- [ ] Compartilhamento de resultado
- [ ] Hub central (Reflexo + Equilíbrio + futuros jogos)

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Veja [CONTRIBUTING.md](CONTRIBUTING.md) para detalhes.

---

## 📄 Licença

MIT — veja [LICENSE](LICENSE) para detalhes.

---

**Versão:** 1.0.0 | Parte da série de puzzles diários junto com o [Reflexo](https://jogo-reflexo.netlify.app)
