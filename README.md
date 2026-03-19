# Neam Skills

A collection of ready-made skills for the [Neam programming language](https://github.com/neam-lang/Neam).

Pick a skill, copy it into your `.neam` file, and attach it to your agent.

---

## Available Skills

| Skill | File | What It Does |
|-------|------|-------------|
| `Calculator` | [calculator](./skills/calculator/) | Add, subtract, multiply, divide, power, square root |
| `WebFetch` | [web-fetch](./skills/web-fetch/) | Fetch a URL with HTTP GET |
| `HTTPRequest` | [http-request](./skills/http-request/) | Make HTTP requests (GET, POST, PUT, DELETE) |
| `FileReader` | [file-reader](./skills/file-reader/) | Read a file from disk |
| `FileWriter` | [file-writer](./skills/file-writer/) | Write content to a file |
| `JSONParser` + `JSONFormatter` | [json-tools](./skills/json-tools/) | Parse and format JSON |
| `GetTimestamp` + `FormatTime` | [timer](./skills/timer/) | Get current time, format timestamps |
| `UUIDGen` | [uuid-gen](./skills/uuid-gen/) | Generate a random UUID |
| `Hasher` + `Base64Encode` | [hasher](./skills/hasher/) | Hash strings, encode to Base64 |
| `TextUpper` + `TextLower` + `TextTrim` | [text-tools](./skills/text-tools/) | Basic text transformations |

---

## How to Use

1. Open any skill folder and copy the `.neam` file contents
2. Paste the skill into your `.neam` program
3. Add the skill name to your agent's `skills` list

```neam
// 1. Paste the skill definition
skill Calculator {
  description: "Perform basic math operations",
  params: [
    { name: "operation", schema: { "type": "string", "description": "add/sub/mul/div/pow/sqrt" } },
    { name: "a", schema: { "type": "number", "description": "First number" } },
    { name: "b", schema: { "type": "number", "description": "Second number" } }
  ],
  impl: fun(operation, a, b) {
    if operation == "add" { return a + b; }
    if operation == "mul" { return a * b; }
    return 0;
  }
}

// 2. Attach it to your agent
agent MathBot {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "You are a math assistant. Use the Calculator skill to solve problems.",
  skills: [Calculator]
}

// 3. Run it
let answer = MathBot.ask("What is 25 multiplied by 4?");
emit answer;
```

---

## Skills

### Calculator
Math operations — add, subtract, multiply, divide, power, square root.
→ [View skill](./skills/calculator/)

### WebFetch
Fetch content from any URL.
→ [View skill](./skills/web-fetch/)

### HTTPRequest
Full HTTP requests with custom method, body, and headers.
→ [View skill](./skills/http-request/)

### FileReader
Read a file from disk. Returns an error message if the file doesn't exist.
→ [View skill](./skills/file-reader/)

### FileWriter
Write text content to a file.
→ [View skill](./skills/file-writer/)

### JSONParser + JSONFormatter
Parse a JSON string into an object, or convert an object back to a JSON string.
→ [View skill](./skills/json-tools/)

### GetTimestamp + FormatTime
Get the current Unix timestamp, or format any timestamp into a readable date.
→ [View skill](./skills/timer/)

### UUIDGen
Generate a random UUID v4. No input needed.
→ [View skill](./skills/uuid-gen/)

### Hasher + Base64Encode
Hash a string (sha256/sha1/md5) or encode it to Base64.
→ [View skill](./skills/hasher/)

### TextUpper + TextLower + TextTrim
Convert text to uppercase, lowercase, or remove extra whitespace.
→ [View skill](./skills/text-tools/)

---

## License

MIT
