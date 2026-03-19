# Calculator Skill

This skill lets your agent do basic math: add, subtract, multiply, divide, raise to a power, or take a square root.

## How to use

Copy the skill block from `calculator.neam` into your `.neam` file, then add `Calculator` to your agent's skills list.

## Example

```
User: What is 7 to the power of 3?

Agent uses: Calculator(operation: "pow", a: 7, b: 3)
Result: 343
```

```
User: What is the square root of 144?

Agent uses: Calculator(operation: "sqrt", a: 144)
Result: 12
```

## Supported operations

| operation | what it does       | example              |
|-----------|--------------------|----------------------|
| `add`     | a + b              | add(10, 5) → 15      |
| `sub`     | a - b              | sub(10, 5) → 5       |
| `mul`     | a × b              | mul(10, 5) → 50      |
| `div`     | a ÷ b              | div(10, 5) → 2       |
| `pow`     | a to the power b   | pow(2, 8) → 256      |
| `sqrt`    | square root of a   | sqrt(81) → 9         |
