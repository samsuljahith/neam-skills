# UUIDGen Skill

This skill generates a random UUID v4 — a unique ID string that looks like `550e8400-e29b-41d4-a716-446655440000`. UUIDs are commonly used as unique identifiers for records, sessions, or files.

## How to use

Copy the skill block from `uuid_gen.neam` into your `.neam` file, then add `UUIDGen` to your agent's skills list.

## Example

```
User: Give me a unique ID for my new record

Agent uses: UUIDGen()
Result: "f47ac10b-58cc-4372-a567-0e02b2c3d479"
```

Every call returns a different UUID — no two will ever be the same.
