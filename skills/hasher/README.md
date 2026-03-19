# Hasher Skill

This file contains two skills for encoding and hashing text.

- **Hasher** — converts a string into a fixed-length hash using SHA-256, SHA-1, or MD5. Useful for checksums and verifying data.
- **Base64Encode** — encodes a string into Base64 format. Useful for embedding data in URLs or HTTP headers.

## How to use

Copy the skill blocks from `hasher.neam` into your `.neam` file, then add `Hasher` and/or `Base64Encode` to your agent's skills list.

## Examples

**Hashing a password:**
```
User: Give me the SHA-256 hash of "hello world"

Agent uses: Hasher(algorithm: "sha256", data: "hello world")
Result: "b94d27b9934d3e08a52e52d7da7dabfac484efe04294e576db37f0b3e8b19d4a"
```

**Base64 encoding:**
```
User: Base64 encode "username:password"

Agent uses: Base64Encode(text: "username:password")
Result: "dXNlcm5hbWU6cGFzc3dvcmQ="
```

## Supported hash algorithms

| algorithm | notes                        |
|-----------|------------------------------|
| `sha256`  | Most secure, recommended     |
| `sha1`    | Older, still widely used     |
| `md5`     | Fast, for checksums only     |
