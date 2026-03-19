# FileReader Skill

This skill lets your agent read any text file from your computer and return its contents as a string.

## How to use

Copy the skill block from `file_reader.neam` into your `.neam` file, then add `FileReader` to your agent's skills list.

## Example

```
User: Read my shopping list at /home/user/shopping.txt

Agent uses: FileReader(path: "/home/user/shopping.txt")
Result: "Apples, Bread, Milk, Eggs"
```

> If the file does not exist, the skill returns a helpful error message instead of crashing.
