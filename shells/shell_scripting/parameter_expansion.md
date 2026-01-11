# Bash Parameter Expansion: Comprehensive Reference

## 1. Introduction

Parameter expansion constitutes a fundamental mechanism in Bash whereby parameters are substituted with their stored values. This operation extends beyond simple value retrieval to encompass sophisticated string manipulation, conditional substitution, and indirection capabilities. The basic form employs the syntax ${parameter}, where parameter may represent a shell variable, positional parameter, or array reference.

## 2. Basic Expansion Syntax

### 2.1 Simple Forms

The elementary expansion syntax takes two forms:

- **$parameter**: Direct expansion without braces
- **${parameter}**: Expansion with explicit braces

Braces become mandatory when parameter is a positional parameter exceeding single digits or when parameter precedes characters that could be interpreted as part of its name. For instance, `${10}` accesses the tenth positional parameter, while `${var}text` prevents ambiguous parsing of `vartext`.

### 2.2 Indirection

When the first character following the opening brace is an exclamation point (!), and parameter is not a nameref, Bash introduces a level of indirection. The value formed by expanding parameter becomes the name of a new parameter, which is then expanded to produce the final result.

**Syntax**: `${!parameter}`

**Example**:
```bash
actual_var="content"
ref="actual_var"
echo "${!ref}"  # Outputs: content
```

For nameref variables, the exclamation point expansion yields the name of the referenced variable rather than performing complete indirect expansion, with exceptions for `${!prefix*}` and `${!name[@]}` patterns.

## 3. Default Value Operators

These operators provide conditional expansion based on whether a parameter is set or null.

### 3.1 Use Default Value

**Syntax**: `${parameter:-word}`

If parameter is unset or null, the expansion of word substitutes for it. The colon's presence determines whether null values trigger substitution. Without the colon (`${parameter-word}`), only unset parameters trigger default substitution.

**Example**:
```bash
unset var
echo "${var:-default}"     # Outputs: default
var=""
echo "${var:-default}"     # Outputs: default
echo "${var-default}"      # Outputs: (empty)
```

### 3.2 Assign Default Value

**Syntax**: `${parameter:=word}`

If parameter is unset or null, word is both assigned to parameter and returned as the expansion result. This operator modifies the parameter's value for subsequent references.

**Example**:
```bash
unset config
echo "${config:=default_config}"  # Assigns and outputs: default_config
echo "$config"                     # Outputs: default_config
```

### 3.3 Use Alternative Value

**Syntax**: `${parameter:+word}`

If parameter is set and non-null, word is substituted; otherwise, a null string results. This inverts the logic of the default value operator.

**Example**:
```bash
var="set"
echo "${var:+alternative}"  # Outputs: alternative
unset var
echo "${var:+alternative}"  # Outputs: (empty)
```

### 3.4 Display Error on Null or Unset

**Syntax**: `${parameter:?word}`

If parameter is set and non-null, its value is expanded; otherwise, word is written to standard error and the shell exits with non-zero status.

**Example**:
```bash
unset required_var
echo "${required_var:?Variable required_var must be set}"
# Error: bash: required_var: Variable required_var must be set
```

## 4. String Length

**Syntax**: `${#parameter}`

This expansion yields the character length of parameter's value. For positional parameters represented by * or @, the expansion returns the count of positional parameters. When parameter is an array subscripted by * or @, the result is the element count.

**Example**:
```bash
text="hello world"
echo "${#text}"        # Outputs: 11
set -- arg1 arg2 arg3
echo "${#@}"           # Outputs: 3
```

## 5. Substring Expansion

**Syntax**: `${parameter:offset:length}`

The expansion extracts a substring from parameter, beginning at offset and extending for length characters. Negative offsets must be separated from the colon by at least one space to avoid confusion with the :- expansion. When length is negative, it represents an offset from the end rather than a character count.

**Example**:
```bash
str="abcdefgh"
echo "${str:2:3}"      # Outputs: cde
echo "${str: -3:2}"    # Outputs: fg (note the space before -)
echo "${str:2:-1}"     # Outputs: bcdefg
```

## 6. Pattern Matching and Removal

### 6.1 Remove Shortest Match from Beginning

**Syntax**: `${parameter#pattern}`

The pattern is expanded and matched against the beginning of parameter's value. The shortest matching pattern is removed from the expansion result.

**Example**:
```bash
path="/usr/local/bin/program"
echo "${path#*/}"      # Outputs: usr/local/bin/program
```

### 6.2 Remove Longest Match from Beginning

**Syntax**: `${parameter##pattern}`

The longest matching pattern is removed from the parameter's beginning.

**Example**:
```bash
path="/usr/local/bin/program"
echo "${path##*/}"     # Outputs: program
```

### 6.3 Remove Shortest Match from End

**Syntax**: `${parameter%pattern}`

The pattern is matched against a trailing portion of parameter's value, and the shortest matching pattern is deleted.

**Example**:
```bash
filename="document.txt.bak"
echo "${filename%.*}"  # Outputs: document.txt
```

### 6.4 Remove Longest Match from End

**Syntax**: `${parameter%%pattern}`

The longest matching trailing pattern is removed.

**Example**:
```bash
filename="document.txt.bak"
echo "${filename%%.*}" # Outputs: document
```

## 7. Pattern Substitution

### 7.1 Replace First Match

**Syntax**: `${parameter/pattern/string}`

The first occurrence of pattern in parameter's value is replaced by string. If string is omitted, pattern matches are deleted.

**Example**:
```bash
text="hello world hello"
echo "${text/hello/goodbye}"  # Outputs: goodbye world hello
echo "${text/hello}"           # Outputs:  world hello
```

### 7.2 Replace All Matches

**Syntax**: `${parameter//pattern/string}`

Using double slashes causes all instances of pattern to be replaced.

**Example**:
```bash
text="hello world hello"
echo "${text//hello/goodbye}"  # Outputs: goodbye world goodbye
```

### 7.3 Anchored Substitution

**Syntax**: 
- `${parameter/#pattern/string}`: Match at beginning
- `${parameter/%pattern/string}`: Match at end

When pattern begins with #, it must match at the beginning of parameter's value; when pattern begins with %, it must match at the end.

**Example**:
```bash
text="prefix_content_suffix"
echo "${text/#prefix/START}"   # Outputs: START_content_suffix
echo "${text/%suffix/END}"     # Outputs: prefix_content_END
```

## 8. Case Modification

### 8.1 Uppercase Conversion

**Syntax**: 
- `${parameter^}`: Convert first character to uppercase
- `${parameter^^}`: Convert all characters to uppercase

The ^ operator converts lowercase letters matching a pattern to uppercase. The single ^ form examines only the first character, while ^^ examines all characters.

**Example**:
```bash
text="hello world"
echo "${text^}"    # Outputs: Hello world
echo "${text^^}"   # Outputs: HELLO WORLD
```

### 8.2 Lowercase Conversion

**Syntax**:
- `${parameter,}`: Convert first character to lowercase
- `${parameter,,}`: Convert all characters to lowercase

The comma operator converts matching uppercase letters to lowercase, with single comma affecting only the first character and double comma affecting all characters.

**Example**:
```bash
text="HELLO WORLD"
echo "${text,}"    # Outputs: hELLO WORLD
echo "${text,,}"   # Outputs: hello world
```

## 9. Name Reference Variables (Namerefs)

Namerefs constitute variables that function as symbolic links to other variables. Operations on nameref variables are redirected to the variables they reference.

**Syntax**: `declare -n nameref=target_variable`

When a nameref variable is assigned, the value is actually stored in the referenced variable. With namerefs, exclamation point expansion yields the name of the referenced variable rather than its value.

**Example**:
```bash
original="initial"
declare -n ref=original
ref="modified"
echo "$original"    # Outputs: modified
echo "${!ref}"      # Outputs: original (the name)
```

## 10. Variable Name Expansion

**Syntax**: `${!prefix*}` or `${!prefix@}`

This expansion returns the names of all variables whose names begin with prefix, separated by the first character of IFS. This differs fundamentally from standard indirection.

**Example**:
```bash
VAR_one="1"
VAR_two="2"
VAR_three="3"
echo "${!VAR*}"    # Outputs: VAR_one VAR_three VAR_two
```

## 11. Parameter Transformation

Bash 4.4 introduced transformation operators of the form `${parameter@operator}`.

### 11.1 Available Operators

The Q operator quotes the parameter for reuse as input, similar to printf %q. The E operator expands C-style backslash sequences, similar to printf %b. The P operator performs prompt string expansion using PS1 rules. The A operator creates an assignment statement similar to declare -p. The lowercase a operator retrieves variable attributes.

**Example**:
```bash
str="hello world"
echo "${str@Q}"           # Outputs: 'hello world'
declare -a arr=(1 2 3)
echo "${arr@A}"           # Outputs: declare -a arr=([0]="1" [1]="2" [2]="3")
echo "${arr@a}"           # Outputs: a
```

## 12. Special Considerations

### 12.1 Word Splitting

Unquoted parameter expansions undergo word splitting, whereby whitespace characters cause the shell to split the expansion into multiple arguments. This necessitates quoting expansions to preserve literal values containing whitespace.

**Example**:
```bash
file="my document.txt"
rm $file              # Attempts to remove 'my' and 'document.txt' separately
rm "$file"            # Correctly removes 'my document.txt'
```

### 12.2 Null vs. Unset

The presence or absence of the colon in conditional operators determines whether the test evaluates both unset and null conditions or only unset conditions. Including the colon treats null values identically to unset variables.

### 12.3 Nesting Limitations

Parameter expansions cannot be nested. Multiple expansion steps require intermediate variable assignment.

**Example**:
```bash
# Incorrect: nested expansion
# result="${${var#prefix}%suffix}"

# Correct: sequential expansion
temp="${var#prefix}"
result="${temp%suffix}"
```

## 13. Practical Applications

### 13.1 Filename Manipulation

```bash
filepath="/usr/local/bin/program.sh"
dirname="${filepath%/*}"          # /usr/local/bin
basename="${filepath##*/}"        # program.sh
filename="${basename%.*}"         # program
extension="${basename##*.}"       # sh
```

### 13.2 Default Configuration Values

```bash
CONFIG_FILE="${1:-/etc/default.conf}"
LOG_LEVEL="${LOG_LEVEL:-INFO}"
TIMEOUT="${TIMEOUT:=30}"
```

### 13.3 String Sanitization

```bash
user_input="  Example   Text  "
trimmed="${user_input#"${user_input%%[![:space:]]*}"}"
trimmed="${trimmed%"${trimmed##*[![:space:]]}"}"
```

### 13.4 Batch File Processing

```bash
for file in *.jpg; do
    mv "$file" "${file%.jpg}.backup.jpg"
done
```

---
