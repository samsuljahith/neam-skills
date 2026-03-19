# JSON Tools Skill

This file contains two skills for working with JSON data.

- **JSONParser** — turns a JSON string into a usable object.
- **JSONFormatter** — converts data back into a clean JSON string.

## How to use

Copy the skill blocks from `json_tools.neam` into your `.neam` file, then add `JSONParser` and/or `JSONFormatter` to your agent's skills list.

## Examples

**Parsing a JSON string:**
```
User: Parse this JSON: {"name": "Alice", "age": 30}

Agent uses: JSONParser(text: "{\"name\": \"Alice\", \"age\": 30}")
Result: { name: "Alice", age: 30 }
```

**Formatting data back to JSON:**
```
User: Give me that data as a JSON string

Agent uses: JSONFormatter(data: "{\"name\": \"Alice\", \"age\": 30}")
Result: "{\"name\":\"Alice\",\"age\":30}"
```
