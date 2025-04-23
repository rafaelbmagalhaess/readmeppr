
# Jogo de Damas

Este documento apresenta um **guia passo a passo** para a construção do jogo de **Damas** utilizando o framework de jogos de tabuleiro desenvolvido com base em padrões de projeto. O objetivo é garantir uma arquitetura modular, extensível e compatível com outros jogos construídos no mesmo ecossistema.

---

## Etapa 1: Entender os Desafios Específicos do Jogo de Damas

Antes de iniciar a implementação, é importante compreender os principais aspectos que definem o jogo de Damas:

### Regras específicas de movimentação e captura
- Movimentos diagonais em casas escuras.
- Captura obrigatória de peças adversárias.
- Possibilidade de múltiplas capturas sequenciais.
- Promoção da peça para “Dama” ao alcançar a última linha do oponente.

### Movimento limitado até promoção
- Peões movimentam-se apenas para frente (diagonal).
- Damas podem se mover para frente e para trás em qualquer distância diagonal.

**Solução geral:** aproveitar o arcabouço do framework com ajustes no Builder, fábricas de peças, movimentos e promoções.

---

## Etapa 2: Criar os Tipos Básicos

### `DamasPieceType`
- **Tipo**: Enum
- **Implementa**: `PieceType`
- **Uso**: Identificar se é um peão ou uma dama.
- **Padrão aplicado**: Flyweight

```java
public enum DamasPieceType implements PieceType {
    PEAO, DAMA;

    @Override
    public String getName() {
        return name();
    }
}
```

### `DamasCellType`
- **Tipo**: Enum
- **Implementa**: `CellType`
- **Uso**: Definir células válidas (geralmente apenas as escuras são usadas)

```java
public enum DamasCellType implements CellType {
    VÁLIDA, INVÁLIDA;

    @Override
    public String getName() {
        return name();
    }
}
```

---

## Etapa 3: Criar a Fábrica Principal do Jogo

### `DamasAbstractFactory`
- **Herda de**: `GameAbstractFactory`
- **Responsabilidades**:
  1. Criar peças iniciais com `DamasGamePieceFactory`
  2. Criar tabuleiro com `DamasBoardBuilder`
  3. Criar os jogadores "Branco" e "Preto"
  4. Registrar o jogo no `GameRegistry`
- **Padrão aplicado**: Abstract Factory

```java
@GameId("Damas")
public class DamasAbstractFactory implements GameAbstractFactory {
    static {
        GameRegistry.register("Damas", new DamasAbstractFactory());
    }

    @Override
    public List<GamePiece> createGamePieces() {
        return new DamasGamePieceFactory().createAllInitialPieces();
    }

    @Override
    public GameBoard createGameBoard() {
        return new GameBoardDirector(new DamasBoardBuilder()).construct(8, 8);
    }

    @Override
    public List<Player> createPlayers() {
        return List.of(new Player("Branco"), new Player("Preto"));
    }
}
```

---

## Etapa 4: Construir o Tabuleiro do Jogo

### `DamasBoardBuilder`
- **Implementa**: `BoardBuilder`
- **Responsável por**:
  1. Criar tabuleiro 8x8
  2. Definir células válidas para o jogo
  3. Posicionar os peões iniciais nas três primeiras linhas de cada jogador
- **Padrão aplicado**: Builder

```java
@Override
public void populatePieces() {
    List<GamePiece> pieces = board.getPieces().getAll();
    // Distribuir os peões nos lugares válidos, como casas escuras das 3 primeiras e últimas linhas
}
```

---

## Etapa 5: Criar as Peças com Regras de Movimento

### `DamasGamePieceFactory`
- **Herda de**: `GamePieceFactory`
- **Responsável por**:
  1. Criar peões e aplicar comportamento com base no tipo
  2. Usar `DamasMoveFactory` para definir as regras de movimentação e captura
- **Padrões aplicados**: Factory Method + Flyweight

```java
@Override
protected GamePiece createGamePiece(PieceType type) {
    DamasPieceType damasType = (DamasPieceType) type;
    Move moveChain = moveFactory.createMoveChain(damasType);
    GamePieceProps props = new GamePieceProps(damasType, moveChain);
    return new GamePiece(props);
}
```

---

## Etapa 6: Criar a Fábrica de Movimentos

### `DamasMoveFactory`
- **Herda de**: `AbstractMoveFactory`
- **Responsável por**:
  1. Definir regras de movimento para peões e damas
  2. Implementar movimentação diagonal simples e captura em salto
  3. Lidar com múltiplas capturas e promoção
- **Padrões aplicados**: Chain of Responsibility + Factory Method

```java
@Override
public Move createMoveChain(DamasPieceType type) {
    return switch (type) {
        case PEAO -> chain(new DiagonalStepForward(), new CaptureMove());
        case DAMA -> chain(new DiagonalSlide(), new MultiCaptureMove());
    };
}
```

---

## Etapa 7: (Opcional) Estender `GameBoard`

### `DamasBoard`
- **Herda de**: `GameBoard`
- **Responsável por**:
  - Validar se capturas estão disponíveis
  - Aplicar promoção ao chegar na última linha
  - Verificar fim de jogo
- **Padrão aplicado**: Template Method

```java
public class DamasBoard extends GameBoard {
    public void checkPromotion(Position position, GamePiece piece) {
        // Lógica de promoção de peão para dama
    }
}
```

---

## Etapa 8: Integração com o Framework

### Passo 1: Registro do jogo
```java
GameRegistry.register("Damas", new DamasAbstractFactory());
```

### Passo 2: Inicialização da partida
```java
GameManager.getInstance().start("Damas", "Branco");
```

> Isso cria o tabuleiro, as peças e inicia o controle via `GameSession` (Facade).

---

## Etapa 9: Visão Geral dos Padrões Utilizados

| Padrão                | Aplicação no Jogo de Damas                   |
|-----------------------|---------------------------------------------|
| **Abstract Factory**  | `DamasAbstractFactory`                      |
| **Builder**           | `DamasBoardBuilder`, `GameBoardDirector`    |
| **Factory Method**    | `DamasGamePieceFactory`, `DamasMoveFactory` |
| **Flyweight**         | `GamePieceProps` compartilhado por tipo     |
| **Chain of Responsibility** | Movimento das peças                   |
| **Template Method**   | Validação e promoção no tabuleiro           |
| **Prototype**         | Clonagem de peças e posições                |
| **Iterator**          | Iteração sobre peças no tabuleiro           |
| **Command**           | Comandos de movimentação e ações            |
| **Singleton**         | `GameManager`                               |
| **Facade**            | `GameSession`                               |

---

## Considerações Finais

Com este guia detalhado, a construção do jogo de Damas no framework se torna organizada, reutilizável e facilmente ajustável para variações de regras. A estrutura baseada em padrões de projeto garante flexibilidade para futuras expansões e integração com outros jogos.

