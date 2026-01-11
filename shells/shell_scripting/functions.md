# Bash Functions: Complete Reference Guide

## 1. Introduction

A bash function is a reusable block of code that groups commands under a single name. Functions execute in the current shell context without creating a new process, making them efficient for organizing scripts and reducing code repetition.

**Benefits:**
- Code reusability and modularity
- Improved script organization
- Reduced code duplication
- Faster execution than external scripts
- Easier maintenance and debugging

---

## 2. Function Definition

Bash provides two syntactically valid methods for defining functions.

### Method 1: Standard Syntax (Recommended)

```bash
function_name() {
    commands
}
```

This is the preferred and most portable syntax. It works in all POSIX-compliant shells.

### Method 2: Using the function Keyword

```bash
function function_name {
    commands
}
```

When using the `function` keyword, parentheses are optional.

### Method 3: Combined Syntax

```bash
function function_name() {
    commands
}
```

All three methods are functionally equivalent.

### Syntax Rules

**Curly Braces:**
- Must be separated from the function body by spaces or newlines
- Opening brace can be on the same line or next line
- Closing brace must be on its own line or preceded by a semicolon

**Single-Line Format:**
```bash
function_name() { command1; command2; command3; }
```
Note: Semicolons are required after each command, including the last one.

**Multi-Line Format:**
```bash
function_name() {
    command1
    command2
    command3
}
```

### Function Names

**Naming Rules:**
- Must not contain `$` character
- Can be any unquoted shell word
- Should be descriptive and meaningful
- Cannot be the same as special builtins in POSIX mode

**Valid Names:**
```bash
my_function
myFunction
my-function
_private_function
calculateSum
```

### Function Definition Location

Functions must be defined before they are called. Definitions can appear:
- At the top of a script
- In configuration files like `~/.bashrc`
- In separate library files that are sourced
- Directly in the terminal for interactive use

---

## 3. Calling Functions

### Basic Invocation

Call a function by writing its name as a command:

```bash
#!/bin/bash

greet() {
    echo "Hello, World!"
}

greet  # Outputs: Hello, World!
```

### Multiple Invocations

Functions can be called multiple times:

```bash
greet
greet
greet  # Calls the function three times
```

### Function Execution Order

```bash
#!/bin/bash

function_a() {
    echo "Function A"
    function_b
}

function_b() {
    echo "Function B"
}

function_a
# Output:
# Function A
# Function B
```

Functions do not need to be declared in any specific order, but they must be defined before being called.

---

## 4. Function Parameters and Arguments

### Positional Parameters

Functions access arguments through positional parameters:

```bash
greet() {
    echo "Hello, $1!"
}

greet "Alice"  # Outputs: Hello, Alice!
```

**Special Parameters:**
- `$1, $2, $3, ...` - Individual arguments
- `$0` - Script name (not function name)
- `$#` - Number of arguments passed
- `$@` - All arguments as separate words
- `$*` - All arguments as a single word
- `$?` - Exit status of last command

### Multiple Arguments

```bash
calculate_sum() {
    echo "First number: $1"
    echo "Second number: $2"
    echo "Sum: $(( $1 + $2 ))"
}

calculate_sum 5 10
```

### Processing All Arguments

**Using `$@` (Recommended):**
```bash
print_all() {
    for arg in "$@"; do
        echo "$arg"
    done
}

print_all one two "three four"
```

**Difference Between `$@` and `$*`:**
- `"$@"` expands to separate strings: `"$1" "$2" "$3"`
- `"$*"` expands to a single string: `"$1 $2 $3"`

### Checking Argument Count

```bash
require_args() {
    if [ $# -lt 2 ]; then
        echo "Error: At least 2 arguments required"
        return 1
    fi
    echo "Processing $# arguments"
}
```

### Shifting Arguments

The `shift` command removes arguments from the beginning:

```bash
process_args() {
    echo "First arg: $1"
    shift
    echo "New first arg: $1"
    shift 2  # Shift by 2 positions
    echo "After shifting 2: $1"
}

process_args a b c d e
```

---

## 5. Return Values and Exit Status

### Return Status

Unlike functions in other languages, bash functions return an exit status (0-255), not arbitrary values.

**Using return:**
```bash
is_valid() {
    if [ "$1" -gt 0 ]; then
        return 0  # Success
    else
        return 1  # Failure
    fi
}

if is_valid 5; then
    echo "Valid"
fi
```

**Return Status Rules:**
- `0` indicates success
- `1-255` indicate various error conditions
- Default return status is the exit status of the last command
- Variable `$?` contains the return status

### Returning Values via Standard Output

To return actual data, use `echo` or `printf` and command substitution:

```bash
get_square() {
    local result=$(( $1 * $1 ))
    echo "$result"
}

value=$(get_square 7)
echo "Square is: $value"
```

### Returning Multiple Values

**Method 1: Multiple echo statements**
```bash
get_stats() {
    echo "10"
    echo "20"
    echo "30"
}

# Capture in array
IFS=$'\n' read -d '' -ra values <<< "$(get_stats)"
```

**Method 2: Using a delimiter**
```bash
get_name_age() {
    echo "Alice:25"
}

result=$(get_name_age)
name="${result%%:*}"
age="${result##*:}"
```

**Method 3: Using global variables (less preferred)**
```bash
calculate() {
    sum=$(( $1 + $2 ))
    product=$(( $1 * $2 ))
}

calculate 5 3
echo "Sum: $sum, Product: $product"
```

---

## 6. Variable Scope

### Global Variables

By default, all variables in bash are global:

```bash
#!/bin/bash

my_var="global"

my_function() {
    my_var="modified"
}

echo "$my_var"    # Outputs: global
my_function
echo "$my_var"    # Outputs: modified
```

### Local Variables

Use `local` to restrict variable scope to the function:

```bash
#!/bin/bash

my_var="global"

my_function() {
    local my_var="local"
    echo "Inside: $my_var"
}

echo "Before: $my_var"  # Outputs: global
my_function             # Outputs: Inside: local
echo "After: $my_var"   # Outputs: global
```

**Key Points:**
- `local` can only be used inside functions
- Local variables hide global variables with the same name
- Local variables are destroyed when the function exits
- Child functions inherit local variables from parent functions

### Multiple Local Variables

```bash
process_data() {
    local name="Alice"
    local age=30
    local city="New York"
    
    echo "$name is $age years old from $city"
}
```

### Local Variable Attributes

```bash
my_function() {
    local -i count=0      # Integer
    local -r constant=10  # Readonly
    local -a array=()     # Array
    local -A assoc=()     # Associative array
}
```

---

## 7. Arrays and Functions

### Passing Arrays to Functions

**Method 1: Expanding array elements (creates copies)**
```bash
print_array() {
    local arr=("$@")
    for element in "${arr[@]}"; do
        echo "$element"
    done
}

my_array=("apple" "banana" "cherry")
print_array "${my_array[@]}"
```

**Method 2: Using name references (pass by reference)**
```bash
modify_array() {
    local -n arr=$1  # Create name reference
    arr+=("new_element")
}

my_array=("apple" "banana")
modify_array my_array  # Pass array name, not expansion
echo "${my_array[@]}"  # Outputs: apple banana new_element
```

### Name References (local -n)

Name references allow functions to work with the original variable:

```bash
#!/bin/bash

update_value() {
    local -n ref=$1  # Create reference to variable
    ref="modified"
}

my_var="original"
update_value my_var
echo "$my_var"  # Outputs: modified
```

**Important Notes:**
- Name references require Bash version 4.3+
- Cannot apply nameref attribute to array variables themselves
- Useful for modifying caller's variables directly

### Returning Arrays

**Method 1: Using name references**
```bash
create_array() {
    local -n result=$1
    result=("one" "two" "three")
}

declare -a my_array
create_array my_array
echo "${my_array[@]}"
```

**Method 2: Using command substitution**
```bash
get_files() {
    find . -type f -name "*.txt"
}

files=($(get_files))
```

---

## 8. Recursion

### Basic Recursion

Functions can call themselves:

```bash
#!/bin/bash

countdown() {
    if [ $1 -le 0 ]; then
        echo "Done!"
        return
    fi
    echo "$1"
    countdown $(( $1 - 1 ))
}

countdown 5
```

### Factorial Example

```bash
factorial() {
    local n=$1
    if [ $n -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $(( n - 1 )))
        echo $(( n * prev ))
    fi
}

result=$(factorial 5)
echo "5! = $result"  # Outputs: 5! = 120
```

### Recursion Depth Limit

The `FUNCNEST` variable controls maximum recursion depth:

```bash
# Set recursion limit
FUNCNEST=100

recursive_func() {
    recursive_func
}

recursive_func  # Will abort after 100 calls
```

**Important:**
- By default, bash places no limit on recursion
- Without a limit, infinite recursion causes stack overflow
- Set `FUNCNEST` to prevent runaway recursion
- The function call stack is limited by available memory

### Tail Recursion

Bash does not optimize tail recursion:

```bash
# Not optimized - still consumes stack space
tail_sum() {
    [ $1 -eq 0 ] && echo $2 && return
    tail_sum $(( $1 - 1 )) $(( $2 + $1 ))
}
```

For performance-critical recursive tasks, consider iterative approaches.

---

## 9. Export and Sourcing

### Exporting Functions

Functions are not automatically available to child processes. Use `export -f`:

```bash
#!/bin/bash

my_function() {
    echo "Hello from function"
}

# Export function to child processes
export -f my_function

# Now available in subshells
bash -c 'my_function'
```

### Viewing Exported Functions

```bash
# List all exported variables and functions
export -p

# List only functions
declare -F

# Show function definition
declare -f function_name
```

### Sourcing Function Libraries

Create reusable function libraries:

**File: mylib.sh**
```bash
#!/bin/bash

utility_function() {
    echo "Utility function"
}

helper_function() {
    echo "Helper function"
}
```

**Using the library:**
```bash
#!/bin/bash

# Source the library
source mylib.sh
# or
. mylib.sh

# Now functions are available
utility_function
helper_function
```

### Function Availability in Different Contexts

**Terminal session:**
```bash
# Define function in current shell
my_func() { echo "test"; }

# Available in current shell
my_func

# NOT available in subshell without export
bash -c 'my_func'  # Error

# Make available with export
export -f my_func
bash -c 'my_func'  # Works
```

### Sourcing vs Executing

**Sourcing (runs in current shell):**
```bash
source script.sh  # Functions remain after script ends
. script.sh       # POSIX equivalent
```

**Executing (runs in subshell):**
```bash
./script.sh      # Functions disappear after script ends
bash script.sh   # Same behavior
```

---

## 10. Signal Handling with Traps

### Basic Trap Syntax

```bash
trap 'commands' SIGNAL
```

### Common Signals

- `EXIT` - Script or function exits
- `ERR` - Command returns non-zero exit status
- `DEBUG` - Before each command
- `RETURN` - Function or sourced script returns
- `SIGINT` - Interrupt signal (Ctrl+C)
- `SIGTERM` - Termination signal
- `SIGHUP` - Hangup signal

### EXIT Trap

Execute cleanup on script exit:

```bash
#!/bin/bash

cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}

trap cleanup EXIT

# Create temporary file
echo "data" > /tmp/tempfile

# Cleanup happens automatically on exit
```

### ERR Trap

Handle errors in functions:

```bash
#!/bin/bash

set -E  # Inherit traps in functions

error_handler() {
    echo "Error occurred in ${BASH_SOURCE[1]} at line ${BASH_LINENO[0]}"
    echo "Failed command: ${BASH_COMMAND}"
}

trap error_handler ERR

failing_function() {
    false  # This will trigger ERR trap
}

failing_function
```

### RETURN Trap

Execute code when function returns:

```bash
#!/bin/bash

function_with_return_trap() {
    trap 'echo "Function returning"' RETURN
    echo "Doing work"
}

function_with_return_trap
```

### Signal Handling in Functions

```bash
#!/bin/bash

long_running_task() {
    trap 'echo "Interrupted"; return 1' SIGINT
    
    for i in {1..100}; do
        echo "Processing $i"
        sleep 1
    done
}

long_running_task
```

### Trap Inheritance

**DEBUG and RETURN traps:**
- Not inherited by default
- Use `set -o functrace` or `shopt -s extdebug` to enable inheritance

```bash
set -o functrace

trap 'echo "Debug: $BASH_COMMAND"' DEBUG

my_function() {
    echo "Inside function"
}

my_function  # Debug trap fires for commands in function too
```

### Removing Traps

```bash
# Remove trap
trap - SIGINT

# Ignore signal
trap '' SIGINT

# Restore default behavior
trap - SIGINT
```

---

## 11. Function Management

### Listing Functions

**List function names:**
```bash
declare -F
```

**Show function names and line numbers:**
```bash
shopt -s extdebug
declare -F
```

**View function definition:**
```bash
declare -f function_name
```

**View all function definitions:**
```bash
declare -f
```

### Unsetting Functions

Remove a function definition:

```bash
unset -f function_name
```

**Verify removal:**
```bash
declare -F function_name  # Should return nothing
```

### Checking if Function Exists

```bash
if declare -F function_name > /dev/null; then
    echo "Function exists"
else
    echo "Function does not exist"
fi
```

### Function Debugging

**Enable debugging:**
```bash
set -x  # Print commands as they execute
```

**Debug specific function:**
```bash
my_function() {
    set -x
    # function code
    set +x
}
```

**Using extdebug:**
```bash
shopt -s extdebug

# Shows source file and line number
declare -F function_name
```

### Finding Function Definition Location

```bash
#!/bin/bash

shopt -s extdebug

my_function() {
    echo "test"
}

# Shows: my_function 5 script.sh
declare -F my_function
```

---

## 12. Best Practices

### Naming Conventions

**Good names:**
```bash
calculate_average()
validate_input()
get_user_info()
is_valid_email()
```

**Avoid:**
```bash
func1()
temp()
x()
```

### Always Use Local Variables

```bash
# Bad: Modifies global scope
calculate() {
    result=$(( $1 + $2 ))
}

# Good: Uses local variables
calculate() {
    local result=$(( $1 + $2 ))
    echo "$result"
}
```

### Validate Input

```bash
divide() {
    if [ $# -ne 2 ]; then
        echo "Error: Two arguments required" >&2
        return 1
    fi
    
    if [ $2 -eq 0 ]; then
        echo "Error: Division by zero" >&2
        return 1
    fi
    
    echo $(( $1 / $2 ))
}
```

### Return Appropriate Exit Codes

```bash
validate_file() {
    local file=$1
    
    if [ ! -e "$file" ]; then
        echo "Error: File does not exist" >&2
        return 1
    fi
    
    if [ ! -r "$file" ]; then
        echo "Error: File not readable" >&2
        return 2
    fi
    
    return 0
}
```

### Use Error Messages on stderr

```bash
error_function() {
    echo "Error: Something went wrong" >&2
    return 1
}
```

### Provide Usage Information

```bash
my_command() {
    if [ $# -eq 0 ]; then
        cat >&2 << EOF
Usage: my_command <arg1> <arg2>

Description:
    Does something useful with the arguments.

Arguments:
    arg1 - First argument description
    arg2 - Second argument description
EOF
        return 1
    fi
    
    # Function implementation
}
```

### Keep Functions Focused

Each function should do one thing well:

```bash
# Bad: Does too much
process_data() {
    validate_input "$1"
    clean_data "$1"
    transform_data "$1"
    save_data "$1"
    send_notification
}

# Good: Single responsibility
validate_input() {
    # Just validation
}

clean_data() {
    # Just cleaning
}
```

### Document Complex Functions

```bash
# Calculates the factorial of a number using recursion
# Arguments:
#   $1 - The number to calculate factorial for (must be >= 0)
# Returns:
#   Prints the factorial result
#   Returns 1 on error, 0 on success
# Example:
#   result=$(factorial 5)
factorial() {
    local n=$1
    
    if [ $n -lt 0 ]; then
        echo "Error: Negative numbers not supported" >&2
        return 1
    fi
    
    if [ $n -le 1 ]; then
        echo 1
        return 0
    fi
    
    local prev=$(factorial $(( n - 1 )))
    echo $(( n * prev ))
}
```

---

## 13. Advanced Topics

### Function Wrappers

Override existing commands:

```bash
# Wrapper for ls with default options
ls() {
    command ls --color=auto -lh "$@"
}

# Call the original ls
command ls
```

**Note:** Use `command` keyword to call the original command inside the wrapper.

### Disabling Builtins

```bash
# Disable a builtin
enable -n echo

# Now bash will use external /bin/echo
echo "test"

# Re-enable builtin
enable echo
```

### Dynamic Function Creation

```bash
# Create function from string
eval "my_func() { echo 'Dynamic function'; }"

my_func  # Outputs: Dynamic function
```

**Warning:** Be extremely cautious with `eval` as it can execute arbitrary code.

### Functions with Redirections

```bash
log_function() {
    echo "This goes to a file"
} > /tmp/output.log

log_function  # Output redirected to file
```

### Functions and Pipelines

```bash
generate_numbers() {
    for i in {1..10}; do
        echo $i
    done
}

# Use function in pipeline
generate_numbers | grep 5
```

### Command Substitution in Functions

```bash
current_time() {
    date +"%Y-%m-%d %H:%M:%S"
}

echo "Current time: $(current_time)"
```

### Functions Calling Other Functions

```bash
validate() {
    [ -n "$1" ]
}

process() {
    if validate "$1"; then
        echo "Processing: $1"
    else
        echo "Invalid input" >&2
        return 1
    fi
}

process "data"
```

### Variable Scoping with Subshells

```bash
outer_function() {
    local var="outer"
    
    (
        var="inner"
        echo "In subshell: $var"  # Outputs: inner
    )
    
    echo "After subshell: $var"  # Outputs: outer
}
```

### Performance Considerations

Functions are faster than external scripts:

```bash
# Slow: Spawns new process for each call
for i in {1..1000}; do
    ./external_script.sh
done

# Fast: No process spawning
for i in {1..1000}; do
    internal_function
done
```

### Function Libraries Pattern

```bash
# lib/utils.sh
#!/bin/bash

utils::validate_email() {
    [[ "$1" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

utils::get_timestamp() {
    date +%s
}

# main.sh
source lib/utils.sh

if utils::validate_email "test@example.com"; then
    echo "Valid email"
fi
```

### Readonly Functions

```bash
my_function() {
    echo "Cannot be modified"
}

# Make function readonly
readonly -f my_function

# Attempting to redefine will fail
my_function() {
    echo "New definition"
}  # Error: my_function: readonly function
```

---
