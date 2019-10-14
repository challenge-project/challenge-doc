# Direction in Challenge API

Direction in Challenge API is a `Text` containing one of the cardinal directions:

- North
- South
- West
- East

If you define any direction parameter as empty `Text`, the used direction is always north.

You can modify the direction simply via these available functions:

- `OppositeDirection(Text _Direction)`
- `ClockwiseDirection(Text _Direction)`: Gives the next clockwise direction.
- `ClockwiseDirection(Text _Direction, Integer _Amount)`: Spins the `_Amount` of times clockwise and returns the final direction.
- `CounterClockwiseDirection(Text _Direction)`: Gives the next counter clockwise direction.
- `CounterClockwiseDirection(Text _Direction, Integer _Amount)`: Spins the `_Amount` of times counter clockwise and returns the final direction.