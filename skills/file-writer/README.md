# FileWriter Skill

This skill lets your agent save text content to a file on disk. If the file already exists, it will be overwritten.

## How to use

Copy the skill block from `file_writer.neam` into your `.neam` file, then add `FileWriter` to your agent's skills list.

## Example

```
User: Save "Hello, world!" to /home/user/greeting.txt

Agent uses: FileWriter(path: "/home/user/greeting.txt", content: "Hello, world!")
Result: "Written to: /home/user/greeting.txt"
```

> Tip: Combine this with FileReader to read, modify, and save files.
