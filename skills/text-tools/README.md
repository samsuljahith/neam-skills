# Text Tools Skill

This file contains three simple skills for cleaning up and transforming text.

- **TextUpper** — converts text to ALL CAPS.
- **TextLower** — converts text to all lowercase.
- **TextTrim** — removes extra spaces from the start and end of text.

## How to use

Copy the skill blocks from `text_tools.neam` into your `.neam` file, then add any of `TextUpper`, `TextLower`, or `TextTrim` to your agent's skills list.

## Examples

**Uppercase:**
```
User: Make "hello world" uppercase

Agent uses: TextUpper(text: "hello world")
Result: "HELLO WORLD"
```

**Lowercase:**
```
User: Make "STOP SHOUTING" lowercase

Agent uses: TextLower(text: "STOP SHOUTING")
Result: "stop shouting"
```

**Trim whitespace:**
```
User: Clean up this text: "   too much space   "

Agent uses: TextTrim(text: "   too much space   ")
Result: "too much space"
```
