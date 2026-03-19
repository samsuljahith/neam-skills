---
name: neam-testing
description: Neam testing patterns — unit tests, assertion functions, test organization, and verification strategies for Neam programs.
origin: neam-skills
---

# Neam Testing Patterns

Patterns for writing reliable, maintainable tests for Neam programs using the built-in `test` construct and assertion functions.

## When to Activate

- Writing tests for Neam functions and agents
- Verifying business logic in Neam code
- Setting up test suites for a Neam project
- Running CI/CD verification for Neam programs

---

## The `test` Block

Neam has built-in test support. Use the `test` keyword to define test blocks, and assertion functions to verify behavior.

```neam
test "addition works" {
  assert_eq(2 + 3, 5);
}

test "string concatenation" {
  let result = "Hello" + " " + "World";
  assert_eq(result, "Hello World");
}
```

Run tests with:

```bash
# Compile and run tests
neamc tests/test_utils.neam -o build/test_utils.neamb
neam-cli build/test_utils.neamb --test

# Or via CMake/CTest in the build directory
cd build && ctest --output-on-failure
```

---

## Assertion Functions

| Function | Description |
|----------|-------------|
| `assert_eq(a, b)` | Assert `a == b` |
| `assert_ne(a, b)` | Assert `a != b` |
| `assert_true(cond)` | Assert condition is truthy |
| `assert_false(cond)` | Assert condition is falsy |
| `assert_throws(fn)` | Assert that `fn` throws an error |

---

## Pattern 1: Unit Tests for Pure Functions

Test pure functions (no agents, no I/O) as the base layer of your test suite.

```neam
fun add(a, b) { return a + b; }
fun multiply(a, b) { return a * b; }
fun clamp(val, min, max) {
  if (val < min) { return min; }
  if (val > max) { return max; }
  return val;
}

test "add: basic arithmetic" {
  assert_eq(add(2, 3), 5);
  assert_eq(add(-1, 1), 0);
  assert_eq(add(0, 0), 0);
}

test "multiply: positive numbers" {
  assert_eq(multiply(3, 4), 12);
  assert_eq(multiply(0, 100), 0);
}

test "clamp: within bounds" {
  assert_eq(clamp(5, 0, 10), 5);
}

test "clamp: below minimum" {
  assert_eq(clamp(-5, 0, 10), 0);
}

test "clamp: above maximum" {
  assert_eq(clamp(15, 0, 10), 10);
}
```

---

## Pattern 2: String Processing Tests

```neam
fun build_greeting(name, title) {
  return "Hello, " + title + " " + name + "!";
}

fun sanitize_input(text) {
  return string_lower(string_trim(text));
}

test "greeting: basic" {
  assert_eq(build_greeting("Smith", "Dr"), "Hello, Dr Smith!");
}

test "greeting: no title" {
  assert_eq(build_greeting("Alice", ""), "Hello,  Alice!");
}

test "sanitize: trim and lowercase" {
  assert_eq(sanitize_input("  HELLO WORLD  "), "hello world");
}

test "sanitize: already clean" {
  assert_eq(sanitize_input("clean"), "clean");
}
```

---

## Pattern 3: Math and Logic Tests

```neam
test "math: absolute value" {
  assert_eq(math_abs(-10), 10);
  assert_eq(math_abs(10), 10);
  assert_eq(math_abs(0), 0);
}

test "math: power" {
  assert_eq(math_pow(2, 8), 256);
  assert_eq(math_pow(10, 0), 1);
}

test "math: clamp range" {
  assert_true(math_clamp(50, 0, 100) == 50);
  assert_true(math_clamp(-1, 0, 100) == 0);
  assert_true(math_clamp(200, 0, 100) == 100);
}
```

---

## Pattern 4: JSON Serialization Tests

```neam
test "json: parse object" {
  let obj = json_parse('{"name": "Alice", "age": 30}');
  assert_eq(obj.name, "Alice");
  assert_eq(obj.age, 30);
}

test "json: stringify round-trip" {
  let original = '{"key": "value"}';
  let obj = json_parse(original);
  let text = json_stringify(obj);
  let reparsed = json_parse(text);
  assert_eq(reparsed.key, "value");
}

test "json: nested object" {
  let obj = json_parse('{"user": {"id": 1, "active": true}}');
  assert_true(obj.user.active);
  assert_eq(obj.user.id, 1);
}
```

---

## Pattern 5: Error / Edge Case Tests

```neam
fun safe_divide(a, b) {
  if (b == 0) { return 0; }
  return a / b;
}

fun validate_email(email) {
  return string_contains(email, "@");
}

test "divide: normal case" {
  assert_eq(safe_divide(10, 2), 5);
}

test "divide: by zero returns zero" {
  assert_eq(safe_divide(10, 0), 0);
}

test "validate_email: valid" {
  assert_true(validate_email("user@example.com"));
}

test "validate_email: invalid — no @" {
  assert_false(validate_email("notanemail.com"));
}

test "validate_email: empty string" {
  assert_false(validate_email(""));
}
```

---

## Pattern 6: Integration Tests with File I/O

```neam
test "file: write and read round-trip" {
  let content = "test content " + time_now();
  file_write_string("./tmp/test_roundtrip.txt", content);
  let read_back = file_read_string("./tmp/test_roundtrip.txt");
  assert_eq(read_back, content);
}

test "file: existence check" {
  file_write_string("./tmp/exists_test.txt", "data");
  assert_true(file_exists("./tmp/exists_test.txt"));
}
```

---

## Pattern 7: Organize Tests by Module

Keep test files mirroring your source structure.

```
tests/
├── test_utils.neam       # Tests for src/utils.neam
├── test_agents.neam      # Tests for src/agents.neam
├── test_knowledge.neam   # Tests for knowledge base integration
└── test_integration.neam # End-to-end integration tests
```

Each test file:

```neam
// tests/test_utils.neam
module tests.utils;
import my.app.utils;

test "util: format_date basic" {
  let ts = 0;
  let result = format_date(ts);
  assert_eq(result, "1970-01-01");
}

test "util: format_date current" {
  let ts = time_now();
  let result = format_date(ts);
  assert_ne(result, "");
}
```

---

## Pattern 8: Checkpoint-Based Integration Testing

Use `checkpoint`/`rewind` in integration test scripts to test failure recovery.

```neam
test "agent pipeline: recovery from failure" {
  checkpoint "start";

  let result = "";
  let success = true;

  // simulate potentially failing operation
  let response = http_get("https://httpbin.org/status/200");
  if (string_length(response) == 0) {
    rewind "start";
    success = false;
  }

  assert_true(success);
}
```

---

## Testing Checklist

- Test happy path (expected inputs, expected outputs)
- Test edge cases (empty strings, zero, null, max values)
- Test error cases (invalid inputs, division by zero, missing files)
- Test each function in isolation before testing compositions
- Mirror your source directory structure in `tests/`
- Run tests as part of every build: `ctest --output-on-failure`

---

## Anti-Patterns

```neam
// Bad: One massive test block testing everything
test "everything" {
  assert_eq(add(1, 2), 3);
  assert_eq(multiply(2, 3), 6);
  assert_true(validate_email("a@b.com"));
  // ...50 more assertions
}

// Good: One test per behavior, clear names
test "add: two positives" { assert_eq(add(1, 2), 3); }
test "multiply: two positives" { assert_eq(multiply(2, 3), 6); }
test "email: valid address" { assert_true(validate_email("a@b.com")); }
```

```neam
// Bad: Vague test name
test "test1" { assert_eq(process("input"), "output"); }

// Good: Descriptive — what it tests and what the expected result is
test "process: transforms input to lowercase output" {
  assert_eq(process("INPUT"), "output");
}
```

__Remember__: Write tests for every public function before writing agents. Agent behavior is hard to test deterministically — focus your unit tests on the pure functions and data transformations that feed into agents. Run `ctest --output-on-failure` in CI on every commit.
