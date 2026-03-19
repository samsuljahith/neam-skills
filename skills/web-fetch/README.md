# WebFetch Skill

This skill lets your agent load a webpage or API endpoint and return its content using a simple HTTP GET request.

## How to use

Copy the skill block from `web_fetch.neam` into your `.neam` file, then add `WebFetch` to your agent's skills list.

## Example

```
User: Get the content from https://api.github.com/zen

Agent uses: WebFetch(url: "https://api.github.com/zen")
Result: "Non-blocking is better than blocking."
```
