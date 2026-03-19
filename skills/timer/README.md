# Timer Skill

This file contains two skills for working with time.

- **GetTimestamp** — returns the current time as a Unix timestamp (number of milliseconds since Jan 1, 1970).
- **FormatTime** — turns a timestamp into a readable date or time string.

## How to use

Copy the skill blocks from `timer.neam` into your `.neam` file, then add `GetTimestamp` and/or `FormatTime` to your agent's skills list.

## Examples

**Getting the current timestamp:**
```
User: What time is it right now (as a timestamp)?

Agent uses: GetTimestamp()
Result: 1711234567890
```

**Formatting a timestamp:**
```
User: What date is timestamp 1711234567890?

Agent uses: FormatTime(timestamp: 1711234567890, format: "%Y-%m-%d")
Result: "2024-03-23"
```

## Common format strings

| format        | example output    |
|---------------|-------------------|
| `%Y-%m-%d`    | 2024-03-23        |
| `%H:%M:%S`    | 14:30:00          |
| `%Y-%m-%d %H:%M:%S` | 2024-03-23 14:30:00 |
