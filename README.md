# NeamSkills

Skills for the [Neam programming language](https://github.com/neam-lang/Neam).

---

## The Main Skill — Start Here

### `neam-programming`

> **If you are new to Neam — import this skill into Claude Code first.**

The `neam-programming` skill teaches Claude the entire Neam language — syntax, agents, RAG, skills, guards, budgets, deployment, and all built-in functions. Once imported, Claude can write and debug Neam programs for you.

**Location:** [`skills/neam-programming/SKILL.md`](./skills/neam-programming/SKILL.md)

**How to use in Claude Code:**
```
/import skills/neam-programming/SKILL.md
```
Then just ask Claude to write Neam code — it knows the full language.

---

## Ready-Made Neam Skills

These are `.neam` files you can copy into your agent. Pick a skill, paste it into your program, and add it to your agent's `skills: []` list.

### Utility
| Skill | File | Description |
|-------|------|-------------|
| `Calculator` | [utility/calculator](./skills/utility/calculator/) | Add, subtract, multiply, divide, power, square root |
| `UUIDGen` | [utility/uuid-gen](./skills/utility/uuid-gen/) | Generate a random UUID v4 |
| `GetTimestamp` `FormatTime` | [utility/timer](./skills/utility/timer/) | Current timestamp, format dates |
| `TextUpper` `TextLower` `TextTrim` | [utility/text-tools](./skills/utility/text-tools/) | Uppercase, lowercase, trim whitespace |
| `Hasher` `Base64Encode` | [utility/hasher](./skills/utility/hasher/) | SHA256/SHA1/MD5 hash, Base64 encode |

### Web
| Skill | File | Description |
|-------|------|-------------|
| `WebFetch` | [web/web-fetch](./skills/web/web-fetch/) | Fetch a URL with HTTP GET |
| `HTTPRequest` | [web/http-request](./skills/web/http-request/) | POST/GET/PUT/DELETE with body and headers |
| `URLBuilder` | [web/url-builder](./skills/web/url-builder/) | Build a URL from base, path, and query string |

### Data
| Skill | File | Description |
|-------|------|-------------|
| `JSONParser` `JSONFormatter` | [data/json-tools](./skills/data/json-tools/) | Parse JSON strings, convert objects to JSON |
| `CSVParser` | [data/csv-parser](./skills/data/csv-parser/) | Parse CSV text into rows |
| `DataCounter` | [data/data-counter](./skills/data/data-counter/) | Count items in a JSON array |

### File
| Skill | File | Description |
|-------|------|-------------|
| `FileReader` | [file/file-reader](./skills/file/file-reader/) | Read a file from disk |
| `FileWriter` | [file/file-writer](./skills/file/file-writer/) | Write content to a file |
| `FileExists` | [file/file-exists](./skills/file/file-exists/) | Check if a file exists |
| `FileCopy` | [file/file-copy](./skills/file/file-copy/) | Copy a file from one path to another |

### Math
| Skill | File | Description |
|-------|------|-------------|
| `UnitConverter` | [math/unit-converter](./skills/math/unit-converter/) | Convert km/miles, kg/lbs, celsius/fahrenheit, meters/feet |
| `FindMax` `FindMin` | [math/statistics](./skills/math/statistics/) | Max and min from a list of numbers |

### Security
| Skill | File | Description |
|-------|------|-------------|
| `PasswordValidator` | [security/password-validator](./skills/security/password-validator/) | Check password strength |
| `HMACSign` | [security/hmac-sign](./skills/security/hmac-sign/) | Generate an HMAC signature |

### Development
| Skill | File | Description |
|-------|------|-------------|
| `LogFormatter` | [development/log-formatter](./skills/development/log-formatter/) | Format log messages with timestamp and level |
| `JSONValidator` | [development/json-validator](./skills/development/json-validator/) | Check if a string is valid JSON |

### Productivity
| Skill | File | Description |
|-------|------|-------------|
| `WordCounter` `CharCounter` | [productivity/word-counter](./skills/productivity/word-counter/) | Count words and characters |
| `DaysFromNow` `TimestampToDate` | [productivity/date-calculator](./skills/productivity/date-calculator/) | Future dates, timestamp conversion |

---

## License

MIT
