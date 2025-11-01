# Chapter 9: Bash - The Glue of the Unix Universe

## The Shell That Runs the World

Bash (Bourne Again SHell) was written by Brian Fox in 1989 as a free replacement for the Bourne shell. It's both a command interpreter and a scripting language. If you use Linux or macOS, you've used Bash, whether you knew it or not.

Bash is not pretty. It's full of quirks, inconsistencies, and footguns. Yet it's indispensable. Every DevOps engineer, system administrator, and backend developer writes Bash scripts. They glue together other programs, automate tasks, and configure systems.

Is Bash a real programming language? That's debatable. It's Turing-complete, so technically yes. But it's unlike any language covered so far. It's not designed for building applications—it's designed for running other applications.

## The Philosophy

Bash inherited Unix philosophy:

**Do one thing well**: Programs should be focused and composable.

**Everything is text**: Programs communicate via text streams.

**Compose small tools**: Pipe output from one program to another.

**Automate everything**: If you do it twice, script it.

Bash itself embodies these principles. It's not about Bash doing the work—it's about Bash orchestrating other tools.

## What It Looks Like

```bash
#!/bin/bash

# Variables (no spaces around =!)
name="Alice"
age=30

# String interpolation
echo "Hello, $name!"
echo "Hello, ${name}!"  # Safer

# Command substitution
current_dir=$(pwd)
file_count=$(ls | wc -l)

# Conditionals
if [ "$age" -gt 18 ]; then
    echo "Adult"
else
    echo "Minor"
fi

# Loops
for file in *.txt; do
    echo "Processing $file"
done

for i in {1..10}; do
    echo "Number $i"
done

while read line; do
    echo "Line: $line"
done < input.txt

# Arrays
fruits=("apple" "banana" "orange")
echo "${fruits[0]}"  # apple
echo "${fruits[@]}"  # all elements

# Functions
greet() {
    local name=$1
    echo "Hello, $name!"
}

greet "Bob"

# Pipes and redirection
cat file.txt | grep "pattern" | sort | uniq > output.txt

# Exit codes
if command; then
    echo "Success"
else
    echo "Failed"
fi
```

Bash syntax is... unique. Spacing matters in unexpected ways. Error messages are cryptic. But it's powerful.

## The Quirks

Bash is full of surprises:

**Spacing matters**:
```bash
# Wrong
if[ "$x" -eq 5 ]  # Syntax error

# Right
if [ "$x" -eq 5 ]  # Note the spaces
```

**Variables need quoting**:
```bash
name="Alice Smith"

# Wrong (word splitting)
echo $name  # Might break with spaces

# Right
echo "$name"  # Always quote variables
```

**`[` vs `[[`**:
```bash
# [ is actually a command (/usr/bin/[)
if [ "$x" = "hello" ]; then
    # ...
fi

# [[ is a bash keyword (more powerful, safer)
if [[ "$x" == "hello" ]]; then
    # ...
fi

# [[ supports pattern matching
if [[ "$file" == *.txt ]]; then
    echo "Text file"
fi
```

**String comparison vs numeric comparison**:
```bash
# String comparison
if [ "$a" = "$b" ]; then

# Numeric comparison
if [ "$a" -eq "$b" ]; then

# Use the wrong one and get surprising results
```

**Return codes are inverted**:
```bash
# 0 = success, non-zero = failure
if command; then
    echo "Command succeeded (returned 0)"
fi

# Check return code
command
if [ $? -eq 0 ]; then
    echo "Success"
fi
```

These quirks trip up beginners and experts alike.

## Pipes and Redirection

The heart of shell scripting:

```bash
# Pipe: output of one command → input of another
cat file.txt | grep "error" | wc -l

# Redirect output to file
echo "Hello" > file.txt        # Overwrite
echo "World" >> file.txt       # Append

# Redirect stderr
command 2> errors.txt          # Stderr to file
command > output.txt 2>&1      # Both stdout and stderr to file
command &> output.txt          # Same (shorter)

# Redirect both to different files
command > output.txt 2> errors.txt

# Pipe stderr
command 2>&1 | grep "error"

# Here documents
cat <<EOF
This is
a multi-line
string
EOF

# Here strings
grep "pattern" <<< "search this string"
```

Mastering redirection is essential for effective shell scripting.

## Built-in Commands vs External Programs

Some commands are built into bash, others are external:

```bash
# Built-ins (fast, part of bash)
cd /tmp
echo "hello"
pwd
exit
source script.sh

# External programs (slower, separate executables)
/bin/ls
/usr/bin/grep
/usr/bin/awk

# Check if something is built-in
type cd    # cd is a shell builtin
type grep  # grep is /usr/bin/grep
```

Built-ins execute faster because they don't spawn a new process.

## Parameter Expansion

Bash has powerful string manipulation:

```bash
name="Alice"

# Length
echo "${#name}"  # 5

# Substring
echo "${name:0:3}"  # Ali

# Default values
echo "${undefined:-default}"  # default
echo "${undefined:=default}"  # Sets undefined to default

# Remove prefix/suffix
filename="document.txt"
echo "${filename%.txt}"     # document (remove suffix)
echo "${filename#doc}"      # ument.txt (remove prefix)

# Replace
text="hello world"
echo "${text/world/universe}"  # hello universe
echo "${text//l/L}"            # heLLo worLd (replace all)

# Case conversion
echo "${name^^}"  # ALICE (uppercase)
echo "${name,,}"  # alice (lowercase)
```

This avoids spawning external programs like `sed` or `awk`.

## Arrays (Sort Of)

Bash arrays are basic but useful:

```bash
# Indexed arrays
fruits=("apple" "banana" "orange")
echo "${fruits[0]}"      # apple
echo "${fruits[@]}"      # all elements
echo "${#fruits[@]}"     # array length

# Loop over array
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Associative arrays (hash maps)
declare -A ages
ages["Alice"]=30
ages["Bob"]=25

echo "${ages["Alice"]}"  # 30

# Loop over associative array
for name in "${!ages[@]}"; do
    echo "$name is ${ages[$name]}"
done
```

Arrays work, but they're not as capable as in Python or JavaScript.

## The `set` Command

Control Bash behavior:

```bash
# Exit on error (very important!)
set -e

# Exit on undefined variable
set -u

# Pipe failures propagate
set -o pipefail

# Combined (common idiom)
set -euo pipefail

# Debugging
set -x  # Print each command before executing
```

`set -euo pipefail` is best practice for robust scripts. It prevents silent failures.

## What Bash Is Best For

**System administration**: Configure servers, manage users, install software.

**Automation**: Automate repetitive tasks.

**CI/CD scripts**: Build, test, and deploy scripts.

**Glue code**: Chain together multiple programs.

**Quick one-liners**: Filter logs, process text, find files.

**Environment setup**: `.bashrc`, `.bash_profile`, dotfiles.

**Orchestration**: Combine command-line tools into workflows.

## What Bash Is Worst For

**Complex logic**: Nested conditions and data structures are painful.

**Portability**: Bash-specific features don't work in sh or other shells.

**Error handling**: Difficult to handle errors gracefully.

**Large programs**: Maintainability suffers beyond a few hundred lines.

**Data processing**: Use Python, Perl, or awk instead.

**Cross-platform**: Bash scripts work on Linux/Mac but not Windows (without WSL).

## Common Patterns

**Check if file exists**:
```bash
if [ -f "file.txt" ]; then
    echo "File exists"
fi

if [ -d "directory" ]; then
    echo "Directory exists"
fi
```

**Safe iteration over files**:
```bash
# Wrong (breaks on spaces)
for file in $(ls *.txt); do

# Right
for file in *.txt; do
    [ -f "$file" ] || continue
    echo "$file"
done
```

**Read input**:
```bash
read -p "Enter your name: " name
echo "Hello, $name!"

# Read password (hidden)
read -sp "Password: " password
```

**Parse command line arguments**:
```bash
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            show_help
            exit 0
            ;;
        -v|--verbose)
            verbose=true
            shift
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
done
```

**Error handling**:
```bash
if ! command; then
    echo "Command failed" >&2
    exit 1
fi

# Or
command || { echo "Failed" >&2; exit 1; }
```

## ShellCheck: The Linter You Need

ShellCheck catches common Bash mistakes:

```bash
# Install
apt-get install shellcheck  # Debian/Ubuntu
brew install shellcheck     # macOS

# Use
shellcheck script.sh
```

ShellCheck will save you hours of debugging. Use it.

## The Ecosystem

**Tools everyone uses**:
- `grep`: Search text
- `sed`: Stream editor (find/replace)
- `awk`: Text processing
- `cut`, `sort`, `uniq`, `wc`: Text utilities
- `find`: Find files
- `curl`/`wget`: Download files
- `jq`: JSON processing
- `git`: Version control

**Modern alternatives**:
- `ripgrep` (`rg`): Faster grep
- `fd`: Faster, simpler find
- `bat`: Better cat
- `exa`: Better ls
- `fzf`: Fuzzy finder

Bash scripts orchestrate these tools.

## Bash vs Zsh vs Fish

**Bash**: Default on most Linux systems. Ubiquitous.

**Zsh**: More features, better autocompletion. Default on macOS since Catalina.

**Fish**: User-friendly, modern syntax. Not POSIX-compliant.

For scripts, use `#!/bin/bash` for portability. For interactive use, Zsh or Fish are nicer.

## The Community

**The Sysadmins**: Live in the terminal. Know bash inside out.

**The DevOps Engineers**: Automate infrastructure with bash scripts.

**The "I Just Googled This" Crowd**: Copy-paste StackOverflow scripts and hope they work.

**The Perfectionists**: Write portable POSIX sh scripts that work everywhere.

**The Modernizers**: Use Python instead of Bash whenever possible.

### Common Phrases
- "It works on my machine"
- "Just pipe it to bash" (said when installing software, questionable practice)
- "Use ShellCheck"
- "Quote your variables!"
- "Just write it in Python"
- "chmod +x script.sh"
- "I forgot to set -e again"

## When to Use Bash vs Python

**Use Bash when**:
- You're orchestrating command-line tools
- The script is simple (<50 lines)
- You need it to work everywhere without dependencies
- You're working with files and processes

**Use Python when**:
- You need complex logic or data structures
- The script will grow beyond 100 lines
- You need error handling
- You're doing actual programming, not gluing commands

Many sysadmins start in Bash, then rewrite in Python when complexity grows.

## Should You Learn Bash?

**Yes**. Not optional.

If you work with servers, deploy software, or use Linux/macOS, you need Bash. At minimum, you need to:
- Navigate the filesystem
- Run commands
- Pipe commands together
- Write simple scripts
- Read others' scripts

You don't need to be an expert, but basic proficiency is essential.

## The Verdict

Bash is not elegant. It's not well-designed by modern standards. It's full of gotchas, inconsistencies, and historical baggage.

But Bash is everywhere. It's the default shell on billions of machines. It's the glue that holds Unix-like systems together. It's how you automate boring tasks, configure servers, and deploy applications.

You don't write applications in Bash. You write scripts that orchestrate applications. You write automation that saves hours of manual work. You write the glue between tools.

Is Bash a great programming language? No. Is it an essential skill? Yes.

Learn enough Bash to be productive. Use `set -euo pipefail`. Quote your variables. Use ShellCheck. And when the script grows beyond 100 lines, consider Python.

Bash won't make you feel clever. But it will make you productive. And in the world of system administration and DevOps, productivity is what matters.

The terminal is the interface to your computer's power. Bash is how you wield that power. Learn it. Respect it. Use it wisely.

---

**Next**: [Chapter 10: Rust - Memory Safety or Your Money Back](10-rust.md)
