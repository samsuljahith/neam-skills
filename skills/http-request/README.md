# HTTPRequest Skill

This skill lets your agent send any kind of HTTP request — GET, POST, PUT, or DELETE — with a custom body and headers.

## How to use

Copy the skill block from `http_request.neam` into your `.neam` file, then add `HTTPRequest` to your agent's skills list.

## Example

```
User: Post a new item to my API

Agent uses: HTTPRequest(
  method: "POST",
  url: "https://myapi.com/items",
  body: "{\"name\": \"apple\", \"count\": 5}",
  headers: "{\"Content-Type\": \"application/json\"}"
)
Result: {"id": "123", "status": "created"}
```

> Tip: If you do not need a body or custom headers, pass an empty string `""` for body and `"{}"` for headers.
