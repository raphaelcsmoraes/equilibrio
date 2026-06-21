# Changelog — Equilíbrio

Todas as mudanças notáveis são documentadas aqui.

Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

## [1.0.0] - 2026-06-20

### Lançamento inicial

- Grade 6×6 com três símbolos: círculo, triângulo e quadrado
- Regra bidimensional: cada linha E cada coluna deve ter 2 de cada símbolo
- Gerador automático de puzzles por backtracking — puzzles nunca se repetem
- Garantia de solução única em todos os tabuleiros
- Dificuldade progressiva por dia da semana (segunda mais fácil → domingo mais difícil)
- Indicadores em tempo real: linhas e colunas ficam verdes ao serem completadas corretamente
- Cronômetro persistente que sobrevive a refreshes da página
- Sistema de dicas com cooldown de 15 segundos
- Rastreamento de sequência de dias consecutivos (streak)
- Tela de conclusão com tempo, streak e contagem regressiva até o próximo puzzle
- Suporte bilíngue PT/EN com detecção automática pelo navegador e botão de troca manual
- Zero dependências externas — funciona offline

[1.0.0]: https://github.com/raphaelcsmoraes/equilibrio/releases/tag/v1.0.0
