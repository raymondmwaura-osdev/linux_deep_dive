# Bash Conditionals Reference Guide

## 1. If Statement Syntax

### Basic If Statement

The if statement executes code when a condition evaluates to true.

```bash
if condition; then
    commands
fi
```

The condition is tested. If it returns an exit status of 0 (success), the commands execute. The keyword `fi` closes the if block.

### If-Else Statement

The if-else statement provides an alternative path when the condition is false.

```bash
if condition; then
    commands_if_true
else
    commands_if_false
fi
```

If the condition returns 0, the first block executes. Otherwise, the else block executes.

### If-Elif-Else Statement

The elif keyword allows testing multiple conditions in sequence.

```bash
if condition1; then
    commands1
elif condition2; then
    commands2
elif condition3; then
    commands3
else
    commands_default
fi
```

Each condition is tested in order. When a condition returns 0, its command block executes and the remaining conditions are skipped. If no condition succeeds, the else block executes (if present).

### Nested If Statements

If statements can be placed inside other if statements.

```bash
if condition1; then
    if condition2; then
        commands
    fi
fi
```

Each if block must close with its own `fi` keyword.

---

## 2. Conditional Test Syntaxes

Bash provides four main syntaxes for writing conditions. Each has different capabilities and compatibility.

### test Command

The `test` command evaluates conditional expressions.

```bash
if test expression; then
    commands
fi
```

Example:
```bash
if test -f file.txt; then
    echo "File exists"
fi
```

### Single Brackets [ ]

Single brackets are equivalent to the `test` command. The brackets are the command itself.

```bash
if [ expression ]; then
    commands
fi
```

Important notes:
- Spaces after `[` and before `]` are required
- The closing `]` is a required argument
- Variables should be quoted to prevent word splitting
- Special characters must be escaped

Example:
```bash
if [ -f file.txt ]; then
    echo "File exists"
fi

if [ "$var" = "value" ]; then
    echo "Match found"
fi
```

String comparison operators `<` and `>` must be escaped:
```bash
if [ "$a" \< "$b" ]; then
    echo "a comes before b"
fi
```

### Double Brackets [[ ]]

Double brackets are a Bash extension providing enhanced testing capabilities.

```bash
if [[ expression ]]; then
    commands
fi
```

Advantages over single brackets:
- No word splitting or globbing of variables (quotes often unnecessary)
- Supports `&&` and `||` operators directly
- Supports pattern matching with wildcards
- Supports regex matching with `=~`
- String comparison operators `<` and `>` work without escaping
- More forgiving with missing variables

Example:
```bash
if [[ -f file.txt ]]; then
    echo "File exists"
fi

if [[ $name == J* ]]; then
    echo "Name starts with J"
fi

if [[ $email =~ ^[a-z]+@[a-z]+\.[a-z]+$ ]]; then
    echo "Valid email format"
fi
```

Pattern matching:
```bash
if [[ $filename == *.txt ]]; then
    echo "Text file"
fi
```

Combining conditions:
```bash
if [[ $age -gt 18 && $age -lt 65 ]]; then
    echo "Working age"
fi
```

**Compatibility note:** Double brackets are not POSIX compliant. Use single brackets for scripts that must run on basic `sh` shells.

### Double Parentheses (( ))

Double parentheses are used for arithmetic evaluation.

```bash
if (( expression )); then
    commands
fi
```

Features:
- Variables do not require `$` prefix
- Supports C-style operators: `<`, `>`, `<=`, `>=`, `==`, `!=`
- Supports arithmetic operations: `+`, `-`, `*`, `/`, `%`, `**`
- Supports increment/decrement: `++`, `--`
- Returns true (0) if result is non-zero
- Returns false (1) if result is zero

Example:
```bash
if (( x > 5 )); then
    echo "x is greater than 5"
fi

if (( x >= 10 && x <= 20 )); then
    echo "x is between 10 and 20"
fi

if (( count++ )); then
    echo "count was non-zero"
fi
```

Arithmetic assignment:
```bash
(( sum = a + b ))
echo $sum
```

---

## 3. Test Operators

### File Test Operators

File test operators check properties of files and directories.

| Operator | Description |
|----------|-------------|
| `-e file` | File exists (any type) |
| `-f file` | File exists and is a regular file |
| `-d file` | File exists and is a directory |
| `-s file` | File exists and has size greater than zero |
| `-r file` | File exists and is readable |
| `-w file` | File exists and is writable |
| `-x file` | File exists and is executable |
| `-L file` | File exists and is a symbolic link |
| `-h file` | File exists and is a symbolic link (same as `-L`) |
| `-b file` | File exists and is a block device |
| `-c file` | File exists and is a character device |
| `-p file` | File exists and is a named pipe |
| `-S file` | File exists and is a socket |
| `-t fd` | File descriptor fd is open and refers to a terminal |
| `-O file` | File exists and is owned by the current user |
| `-G file` | File exists and is owned by the current group |

Example:
```bash
if [ -f config.txt ]; then
    echo "Config file exists"
fi

if [ -r data.csv ] && [ -w data.csv ]; then
    echo "File is readable and writable"
fi
```

### Binary File Comparison Operators

These operators compare two files.

| Operator | Description |
|----------|-------------|
| `file1 -nt file2` | file1 is newer than file2 |
| `file1 -ot file2` | file1 is older than file2 |
| `file1 -ef file2` | file1 and file2 refer to the same file |

Example:
```bash
if [ backup.tar -ot original.tar ]; then
    echo "Backup is outdated"
fi
```

### String Comparison Operators

String operators compare or test string values.

| Operator | Description |
|----------|-------------|
| `=` or `==` | Strings are equal |
| `!=` | Strings are not equal |
| `<` | String1 sorts before string2 (lexicographically) |
| `>` | String1 sorts after string2 (lexicographically) |
| `-z string` | String is empty (zero length) |
| `-n string` | String is not empty (non-zero length) |

Example with single brackets:
```bash
if [ "$name" = "Alice" ]; then
    echo "Hello Alice"
fi

if [ -z "$var" ]; then
    echo "Variable is empty"
fi

if [ "$a" \< "$b" ]; then
    echo "a comes before b alphabetically"
fi
```

Example with double brackets:
```bash
if [[ $name == "Alice" ]]; then
    echo "Hello Alice"
fi

if [[ -z $var ]]; then
    echo "Variable is empty"
fi

if [[ $a < $b ]]; then
    echo "a comes before b alphabetically"
fi
```

### Numeric Comparison Operators

Numeric operators compare integer values. These are used with `[ ]` or `[[ ]]`.

| Operator | Description |
|----------|-------------|
| `-eq` | Equal to |
| `-ne` | Not equal to |
| `-lt` | Less than |
| `-le` | Less than or equal to |
| `-gt` | Greater than |
| `-ge` | Greater than or equal to |

Example:
```bash
if [ $count -eq 10 ]; then
    echo "Count is 10"
fi

if [ $age -ge 18 ]; then
    echo "Adult"
fi
```

**Important:** With `(( ))`, use arithmetic operators instead:

| Operator | Description |
|----------|-------------|
| `==` | Equal to |
| `!=` | Not equal to |
| `<` | Less than |
| `<=` | Less than or equal to |
| `>` | Greater than |
| `>=` | Greater than or equal to |

Example:
```bash
if (( count == 10 )); then
    echo "Count is 10"
fi

if (( age >= 18 )); then
    echo "Adult"
fi
```

### Logical Operators

Logical operators combine multiple conditions.

**With single brackets `[ ]`:**

| Operator | Description |
|----------|-------------|
| `!` | Logical NOT |
| `-a` | Logical AND |
| `-o` | Logical OR |

Example:
```bash
if [ $x -gt 5 -a $x -lt 10 ]; then
    echo "x is between 5 and 10"
fi

if [ ! -f file.txt ]; then
    echo "File does not exist"
fi
```

**With double brackets `[[ ]]`:**

| Operator | Description |
|----------|-------------|
| `!` | Logical NOT |
| `&&` | Logical AND |
| `||` | Logical OR |

Example:
```bash
if [[ $x -gt 5 && $x -lt 10 ]]; then
    echo "x is between 5 and 10"
fi

if [[ ! -f file.txt ]]; then
    echo "File does not exist"
fi
```

**With double parentheses `(( ))`:**

| Operator | Description |
|----------|-------------|
| `!` | Logical NOT |
| `&&` | Logical AND |
| `||` | Logical OR |

Example:
```bash
if (( x > 5 && x < 10 )); then
    echo "x is between 5 and 10"
fi
```

**Between test commands (shell-level):**

You can also chain test commands using shell operators:

```bash
if [ -f file.txt ] && [ -r file.txt ]; then
    echo "File exists and is readable"
fi

if [ $x -gt 10 ] || [ $y -gt 10 ]; then
    echo "At least one value exceeds 10"
fi
```

---

## 4. Best Practices

### Choosing the Right Syntax

**Use `[[ ]]` for:**
- String comparisons
- File tests
- Pattern matching
- General conditionals in Bash scripts

**Use `(( ))` for:**
- Arithmetic comparisons
- Mathematical operations
- Counter variables

**Use `[ ]` only when:**
- POSIX compliance is required
- The script must run on basic `sh` shells

### Spacing Requirements

**Single and double brackets require spaces:**
```bash
# Correct
if [ $x -eq 5 ]; then

# Incorrect
if [$x -eq 5]; then
```

**Double parentheses do not require internal spaces (but may have them):**
```bash
# Both correct
if (( x == 5 )); then
if ((x==5)); then
```

### Quoting Variables

**With single brackets, always quote variables:**
```bash
if [ "$var" = "value" ]; then
    echo "Match"
fi
```

This prevents errors when variables contain spaces or are empty.

**With double brackets, quoting is often unnecessary:**
```bash
if [[ $var == value ]]; then
    echo "Match"
fi
```

However, quoting is still good practice.

### Testing for Empty Variables

Use `-z` to test if a variable is empty:
```bash
if [ -z "$var" ]; then
    echo "Variable is empty"
fi
```

Use `-n` to test if a variable is not empty:
```bash
if [ -n "$var" ]; then
    echo "Variable is not empty"
fi
```

### Checking Command Success

You can test commands directly in if statements:
```bash
if grep -q "pattern" file.txt; then
    echo "Pattern found"
fi

if command -v python3 > /dev/null; then
    echo "Python 3 is installed"
fi
```

Commands return 0 for success and non-zero for failure.

### Exit Status

The if statement tests the exit status of commands:
- 0 means true (success)
- Non-zero means false (failure)

This is opposite to many programming languages where 0 is false.

### Combining Multiple Conditions

**Example combining file and numeric tests:**
```bash
if [[ -f data.txt && $(wc -l < data.txt) -gt 100 ]]; then
    echo "File exists and has more than 100 lines"
fi
```

**Example with complex logic:**
```bash
if [[ $age -ge 18 && $age -le 65 ]] || [[ $status == "retired" ]]; then
    echo "Eligible"
fi
```

### Common Errors to Avoid

**Error 1: Missing spaces in single brackets**
```bash
# Wrong
if [$x -eq 5]; then

# Correct
if [ $x -eq 5 ]; then
```

**Error 2: Using arithmetic operators with single brackets**
```bash
# Wrong
if [ $x > 5 ]; then

# Correct
if [ $x -gt 5 ]; then
# Or
if (( x > 5 )); then
```

**Error 3: Unquoted variables with spaces**
```bash
filename="my file.txt"

# Wrong
if [ -f $filename ]; then

# Correct
if [ -f "$filename" ]; then
```

**Error 4: Comparing strings with numeric operators**
```bash
# Wrong
if [ "10" -gt "9" ]; then  # This works but compares as numbers

# For string comparison
if [[ "10" > "9" ]]; then  # Lexicographic comparison
```

**Error 5: Using `=` instead of `-eq` for numbers in brackets**
```bash
# Unreliable (string comparison)
if [ $x = 5 ]; then

# Correct (numeric comparison)
if [ $x -eq 5 ]; then
```

---
