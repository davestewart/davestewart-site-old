---
title: A Web Developer's Guide to the Command Line
description: A comprehensive guide to mastering the Unix command line for web developers—from fundamental concepts to advanced techniques, customization, and shell selection.
date: 2025-12-18
---

# A Web Developer's Guide to the Command Line

## Intro

If you're a web developer, you probably spend most of your time in your IDE clicking through GUIs, and running the occasional `npm install`. But every time you need to batch-rename files, debug a deployment script, or automate something tedious, you're probably Googling or asking AI and copying and pasting commands you don't understand.

I was the same, and these days increasingly leaning on Claude to do the work for me, but I reached an inflection point where I realized I needed to understand the command line better if I wanted to be more effective and autonomous.

**So that's what this guide is for.**

It's the result of a multiple rounds of questioning and editing between Claude and I, going from basics to advanced topics, with ample context, examples, and explanations; assuming nothing, whilst attempting to be exhaustive.

You will probably recognise many of the commands and patterns, even if you're not familiar with them. As such, this guide aims to be a one-stop-shop for web developers wanting to get comfortable and confident with the command line.

By the end, you won't be a Unix wizard, but you'll be comfortable enough to stop googling "how to recursively delete node_modules" and start actually understanding what `find . -name "node_modules" -type d -prune -exec rm -rf {} +` does (and why it's better than `rm -rf **/node_modules`).

Let's dive in.

## Contents

<NavToc 
  type="list"
  prompt="The basics"
  from="prerequisites"
  to="getting-help">
</NavToc>

<NavToc 
  type="list"
  prompt="Up and running"
  from="real-world-patterns"
  to="simple-scripting">
</NavToc>

<NavToc 
  type="list"
  prompt="Appendices"
  from="users-permissions-and-security"
  to="shells-and-shell-frameworks">
</NavToc>

<NavToc 
  type="list"
  prompt="Wrapping up"
  from="where-to-go-from-here"
  to="quick-reference">
</NavToc>

## Prerequisites

Before we dive into commands and pipelines, let's cover the absolute basics.

<NavToc type="list" section="prerequisites" level="3"></NavToc>

### What is a Shell?

When you open Terminal (macOS) or a terminal emulator (Linux), you're not directly talking to the operating system. Instead, you're talking to a **shell** - a program that interprets your text commands and executes them. Think of it as a translator between you and the computer.

The most common shell is **bash** (Bourne Again Shell), though macOS now defaults to **zsh** (Z Shell). They're very similar, and everything in this guide works in both unless noted otherwise.

### Opening Your Terminal

**On macOS:** Press `Cmd+Space` and type "Terminal", or find it in Applications → Utilities

**On Linux:** Usually `Ctrl+Alt+T`, or search for "Terminal" in your applications

You'll see a prompt (something like `dave@computer:~$`) waiting for you to type commands.

### Your First Commands

When your terminal opens, you're in your **home directory** - your personal space on the computer. Let's learn to move around:

```bash
pwd                 # Print Working Directory - shows where you are
                    # /home/dave or /Users/dave

ls                  # LiSt - shows files in current directory

cd Documents        # Change Directory - moves you into the Documents folder
pwd                 # now you're in /home/dave/Documents

cd ..               # go up one level (back to home)
cd                  # with no arguments, goes to home directory
```

Now you know where you are (`pwd`), what's here (`ls`), and how to move around (`cd`). That's enough to start exploring!

## The Unix Filesystem

<NavToc type="list" section="the-unix-filesystem" level="3"></NavToc>

### Key Concepts

**The PATH:**

When you type a command like `npm` or `git`, the shell searches for an executable file with that name in a list of directories called your PATH. You can see it with:

```bash
echo $PATH
# /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/home/dave/bin
```

The shell searches these directories in order, left to right. That's why you can type `ls` instead of `/bin/ls` - the shell finds it for you. Later, we'll cover how to add your own directories to the PATH.

**Absolute vs relative paths:**

```bash
# absolute - starts with /
/home/dave/projects/myapp
/usr/local/bin/node

# relative - from current location
projects/myapp      
../other-project    # goes up one level then into other-project
./script.sh         # explicit current directory
```

**Directory shortcuts:**

```bash
.                   # current directory
..                  # parent directory
-                   # previous directory

cd ..               # up one level
cd ../..            # up two levels
cd -                # go back to previous directory
```

**Your home directory** is where you live:

```bash
~                   # shorthand for home
$HOME               # environment variable

cd                  # with no args, goes home
cd ~                # explicit
cd ~/projects       # relative to home
```

The key insight: everything is a file (even devices and processes), and the filesystem is one unified tree. You navigate it with paths, either absolute from `/` or relative from where you are. Your home `~` is your personal space, `/usr` is shared programs, `/etc` is configuration, and `/tmp` is scratch space.

**Directory shortcuts:**

```bash
.                   # current directory
..                  # parent directory
-                   # previous directory

cd ..               # up one level
cd ../..            # up two levels
cd -                # go back to previous directory
```

**Your home directory** is where you live:

```bash
~                   # shorthand for home
$HOME               # environment variable

cd                  # with no args, goes home
cd ~                # explicit
cd ~/projects       # relative to home
```

The key insight: everything is a file (even devices and processes), and the filesystem is one unified tree. You navigate it with paths, either absolute from `/` or relative from where you are. Your home `~` is your personal space, `/usr` is shared programs, `/etc` is configuration, and `/tmp` is scratch space.

### The Hierarchy

Everything starts at `/` (root). The structure is standardized across Unix systems, and understanding it helps you navigate and troubleshoot effectively.

**System directories:**

```bash
/bin                # essential binaries (ls, cp, cat)
/sbin               # system binaries (admin tools, requires root)
/usr                # user programs and data
  /usr/bin          # most user commands (git, node, npm)
  /usr/local        # locally installed software (homebrew on macOS)
  /usr/lib          # libraries for programs
/lib                # essential system libraries
/etc                # configuration files (system-wide)
/opt                # optional/add-on software packages
/tmp                # temporary files (cleared on reboot)
/var                # variable data (logs, caches, databases)
  /var/log          # log files
  /var/tmp          # temp files that survive reboot
```

**User directories:**

```bash
/home               # user home directories on Linux
  /home/dave        # your personal space
/Users              # user home directories on macOS
  /Users/dave
/root               # root user's home directory
```

**System resources:**

```bash
/dev                # device files (hardware as files)
  /dev/null         # discard output
  /dev/sda1         # hard drive partition
/proc               # process/system info (not real files)
/sys                # system/kernel info
/mnt                # mount points for temporary filesystems
/media              # removable media (USB drives, etc)
```

### Special Locations

Important directories you should know about:

```bash
~/.ssh/             # SSH keys and config
~/.config/          # modern config location
~/.local/           # user-installed programs/data
  ~/.local/bin/     # user binaries (add to PATH)

/tmp/               # ephemeral (cleared on reboot)
/var/log/           # system logs
/etc/               # system configuration
  /etc/hosts        # hostname mappings
  /etc/profile      # system-wide profile

# macOS specific
~/Library/          # macOS user data/config
/Applications/      # macOS apps
/System/            # macOS system (don't touch)
```

### Hidden Files and Dotfiles

Files starting with `.` are hidden:

```bash
.bashrc                 # config files
.git                    # hidden directories
.env                    # environment variables

ls -a                   # show them
rm -rf .*               # DANGEROUS - includes . and ..
```

Configuration lives in `~/.config/` on modern systems:

```bash
~/.config/
  nvim/
  git/
```

### Symlinks

Symbolic links (symlinks) are shortcuts to other files or directories. Think of them like aliases - they point to another location without copying the actual data.

**The order matters!** Think of it like `cp source destination`:

```bash
# ln -s ORIGINAL NEW_LINK
# create a symlink: ln -s WHERE_IT_POINTS NAME_OF_LINK
ln -s /path/to/original /path/to/link
#     ^^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^
#     what already exists  what you're creating

# Real example: link to a binary so it's in your PATH
ln -s /usr/local/bin/node ~/bin/node
#     ^^^^^^^^^^^^^^^^^^^ the actual node binary (already exists)
#                         ^^^^^^^^^ the shortcut you're creating

# Link a directory from Dropbox
ln -s ~/Dropbox/projects ~/projects
#     ^^^^^^^^^^^^^^^^^^ existing Dropbox folder
#                        ^^^^^^^^^^ new shortcut in your home directory
# Now you can cd ~/projects instead of cd ~/Dropbox/projects
```

**Tip:** Read it as "link the first thing to the second location" or "make the second thing point to the first thing"

```bash
# view symlinks - they show with an arrow
ls -l
# lrwxr-xr-x  1 dave  staff  12 Dec 18 10:00 link -> original
#                                                    ^^^^^ shows where it points

# remove the link (doesn't affect the original!)
rm link             # removes link only
unlink link         # same thing

# if you accidentally delete the original, the link becomes "broken"
# the link still exists but points to nothing
```

**Why use symlinks?**
- Keep files organized in one place but access them from multiple locations
- Switch between different versions of something (like node versions)
- Work around programs that expect files in specific locations

### Navigation Commands

Basic commands for moving around and viewing your location:

```bash
pwd                 # print working directory (see where you are)

cd path             # change directory
ls                  # list files
  ls -l             # long format (permissions, owner, size, date)
  ls -a             # show hidden files (start with .)
  ls -la            # combination
  ls -lh            # human-readable sizes (KB, MB)
  ls -lt            # sort by modification time
  ls -ltr           # reverse (oldest first)

tree                # show directory tree (install: brew install tree or apt install tree)
  tree -L 2         # only 2 levels deep
  tree -d           # directories only
```

### File Operations

Creating, copying, moving, and deleting files and directories:

```bash
# create
touch file.txt      # create empty file or update timestamp
mkdir dir           # create directory
mkdir -p path/to/dir # create parents as needed

# copy
cp source dest      # copy file
cp -r dir1 dir2     # copy directory recursively
cp file{,.bak}      # quick backup: file → file.bak

# move/rename
mv old new          # rename/move
mv file dir/        # move into directory

# delete
rm file             # delete file
rm -r dir           # delete directory recursively
rm -rf dir          # force delete (no confirmation)
rmdir dir           # delete empty directory only

# view
cat file            # dump entire file
less file           # paginated view (q to quit)
head file           # first 10 lines
head -n 20 file     # first 20 lines
tail file           # last 10 lines
tail -f file        # follow (watch for changes)
```

### Finding Files

Locating files and commands on your system:

```bash
# find by name
find . -name "*.js"
find /usr -name node

# find by type
find . -type f          # files only
find . -type d          # directories only

# find by time
find . -mtime -7        # modified in last 7 days
find . -mtime +30       # modified more than 30 days ago

# find and execute
find . -name "*.tmp" -delete
find . -name "*.js" -exec grep "TODO" {} \;

# locate (faster but uses database)
locate file.txt         # searches entire system instantly
# locate uses a pre-built database of all files on your system
# it's MUCH faster than find but the database needs updating
updatedb                # update locate database (usually runs daily automatically, requires sudo)

# which - find command in PATH
which node
which npm
```

### Disk Usage

Checking available space and measuring directory sizes:

```bash
# disk space
df -h               # disk free (all mounted filesystems)
df -h .             # just current filesystem

# directory sizes
du -h               # disk usage (current directory)
du -sh *            # summary of each item
du -sh ~/projects   # size of specific directory
du -h --max-depth=1 # one level deep

# sort by size
du -sh * | sort -h  # sort -h handles human-readable sizes (KB, MB, GB)
```

## Command Line Fundamentals

At its core, the command line is about running programs with arguments: `command arg1 arg2`. Arguments are just strings separated by spaces, though you can use quotes when you need spaces inside an argument like `git commit -m "my message"`. You can also escape spaces with a backslash: `cd My\ Documents` (though quotes are usually clearer: `cd "My Documents"`).

Commands can take **options** (flags) which typically start with `-` for short form like `-a` or `--` for long form like `--all`. You can often combine short flags together: `-la` is the same as `-l -a`.

### Streams and Redirection

**What are streams?** Think of streams as channels for data flow. Every command has three standard streams - they're like pipes that data flows through:

- **stdin (0)** - standard input - where the command reads data from (usually your keyboard)
- **stdout (1)** - standard output - where the command writes normal output (usually your screen)
- **stderr (2)** - standard error - where the command writes error messages (also usually your screen)

By default, you type into stdin, and both stdout and stderr print to your terminal. But you can redirect these streams:

**Basic redirection:**

- `command > file.txt` - redirect stdout to a file (overwrites)
- `command >> file.txt` - append stdout to a file (doesn't overwrite)
- `command < file.txt` - read stdin from a file

**Redirecting errors:**

- `command 2> errors.log` - redirect stderr (errors) to a file
- `command 2>&1` - redirect stderr to wherever stdout is going
- `command &> file.txt` - redirect both stdout and stderr to a file (newer syntax)

**Examples:**

The real power comes from **pipes** using `|`, which connect one command's stdout to another's stdin:

```bash
cat file.txt | grep "search" | wc -l
```

This reads a file with `cat` (concatenate/display), filters lines with `grep` (search for patterns), and counts them with `wc -l` (word count, -l for lines). Each command runs concurrently, processing data as it flows through.

### Chaining and Control

You can run commands sequentially with `;` (runs regardless of success):

```bash
cd project; npm install; npm run build
```

For conditional execution, use `&&` (runs next only if previous succeeds) or `||` (runs next only if previous fails). This lets you do things like:

```bash
npm test && npm publish
command || echo "failed"
```

### Exit Codes

**How does `&&` know if a command succeeded?** Every command returns an exit code when it finishes:
- `0` means success
- Any other number (1, 2, 127, etc.) means failure

You can see the last command's exit code with `$?`:

```bash
npm test
echo $?  # shows 0 if tests passed, 1 if they failed

ls /nonexistent
echo $?  # shows 2 (error)
```

This is what `&&` and `||` actually check:

```bash
# runs second command only if first returns 0
command && echo "success"

# runs second command only if first returns non-zero
command || echo "failed"

# combine them
command && echo "success" || echo "failed"
```

Some commands have specific exit codes with meanings:
- `grep`: 0=found, 1=not found, 2=error
- Most commands: 0=success, 1=general error, 2=misuse, 127=command not found

```bash
grep -q "pattern" file.txt && process_it || skip_it
```

> **For scripts:** The `set -e` option makes the shell exit immediately on any command failure, which is crucial for automation. You can temporarily allow failures with `command || true`. We'll cover this more in the [Simple Scripting](#error-handling) section.

### Background Processes

Normally, when you run a command, the shell waits for it to finish before giving you another prompt. The command runs in the **foreground** - you can't type new commands until it's done.

But sometimes you want to start something and keep working - like running a dev server while you edit code. That's what **background** processes are for.

The `&` operator runs a command **in the background**:

```bash
npm run dev &
# [1] 12345  (job number and process ID)
# shell returns immediately, you can keep working
```

Now the command runs independently in the background while you can type new commands in the foreground. The process continues until it finishes or you close the terminal.

**It's not nested** - you have one foreground process (what you're currently typing/interacting with) and multiple background processes running simultaneously.

**Running multiple things at once:**

```bash
command1 & command2 & command3 &
#        ^          ^          ^ each & puts that command in background
```

This starts all three commands simultaneously in the background. The trailing `&` on command3 is necessary - without it, command3 would run in the foreground and you'd have to wait for it.

This is useful for:
- Running a server while you work
- Processing multiple files at the same time
- Starting several services that don't depend on each other

> **Note:** Background processes still print to your terminal, which can be messy. You might want to redirect their output: `npm run dev > dev.log 2>&1 &`

We'll cover more advanced job control (pausing, resuming, bringing to foreground) in the [Advanced Techniques](#job-control) section.

### Substitution

Command substitution with `$(command)` lets you use one command's output as another's argument:

```bash
echo "Today is $(date)"  # echo prints text to stdout
rm $(find . -name "*.tmp")
```

The older backtick syntax `` `command` `` does the same thing but is less flexible.

Process substitution with `<(command)` creates a temporary file-like input:

```bash
diff <(ls dir1) <(ls dir2)
```

This compares directory listings without creating actual files.

### Variables and Environment

**Variables** let you store values and reuse them:

```bash
name="World"
echo "Hello, $name"  # Hello, World

# use braces when ambiguous
echo "${name}s"      # Worlds
```

**Environment variables** are special variables that affect how programs behave. They're passed to every command you run:

```bash
export EDITOR=vim    # sets your default text editor
export NODE_ENV=production
```

Once exported, every program you run can see these variables. For a single command, you can set variables inline without exporting:

```bash
PORT=3000 npm start  # PORT is only available to npm
```

**Built-in environment variables** you should know:

- `$PATH` - directories where the shell looks for commands ([see Key Concepts](#the-path))
- `$HOME` - your home directory (same as `~`)
- `$USER` - your username
- `$SHELL` - your current shell
- `$PWD` - current working directory

```bash
echo $PATH           # see where commands are found
echo $HOME           # /home/dave
echo "I'm $USER"     # I'm dave
```

**Wildcards** (also called globs) expand to matching filenames:

- `*` matches anything: `ls *.txt` (all .txt files)
- `?` matches one character: `test?.txt` (test1.txt, test2.txt)
- `[abc]` matches any of those: `[abc]*.txt` (files starting with a, b, or c)
- `{a,b}` brace expansion: `cp file.{js,ts} backup/` (copies both .js and .ts versions)

So `rm *.tmp` deletes all .tmp files, and `cp *.{js,ts} backup/` copies both js and ts files.

### Substitution

Command substitution with `$(command)` lets you use one command's output as another's argument:

```bash
echo "Today is $(date)"  # date is a command that prints the current date
rm $(find . -name "*.tmp")
today=$(date +%Y-%m-%d)  # store in a variable
echo "Filename: report-$today.pdf"
```

The older backtick syntax `` `command` `` does the same thing but is less flexible.

Process substitution with `<(command)` creates a temporary file-like input:

```bash
diff <(ls dir1) <(ls dir2)
```

This compares directory listings without creating actual files.

### Advanced Patterns

You can group commands with parentheses to run them in a **subshell** (a new shell process):

```bash
(cd temp && npm install)  # doesn't change your current directory
```

The `cd` only affects the subshell, not your main shell. When the subshell exits, you're still in your original directory.

Or with braces for the current shell:

```bash
{ command1; command2; }
```

### Converting Pipe Output to Arguments (xargs)

**The problem:** Pipes are amazing, but they have a limitation. Look at this:

```bash
find . -name "*.log"
# outputs:
# ./app.log
# ./debug.log
# ./error.log
```

You might think you can pipe this to `rm`:

```bash
find . -name "*.log" | rm  # DOESN'T WORK
```

But `rm` doesn't read from stdin - it expects filenames as arguments like `rm file1 file2 file3`. The pipe gives `rm` nothing to work with.

**This is what xargs solves.** It converts stdin (lines of text) into command arguments:

```bash
find . -name "*.log" | xargs rm
# xargs turns this into: rm ./app.log ./debug.log ./error.log
```

**Basic usage:**

```bash
# find and delete
find . -name "*.tmp" | xargs rm

# find and grep inside files
find . -name "*.js" | xargs grep "TODO"

# count lines in multiple files
ls *.txt | xargs wc -l
```

**The -I flag** lets you control where arguments go:

```bash
# copy files with specific naming
ls *.txt | xargs -I {} cp {} {}.backup
# turns into: cp file1.txt file1.txt.backup, cp file2.txt file2.txt.backup, etc.

# mv with renaming
find . -name "*.jpeg" | xargs -I {} mv {} {}.jpg
```

**The -P flag** runs commands in parallel:

```bash
# process 4 files at once
find . -name "*.txt" | xargs -P 4 -I {} process {}
```

**Why not just use -exec?** You can! `find` has its own `-exec` option:

```bash
find . -name "*.log" -exec rm {} \;   # one rm per file (slow)
find . -name "*.log" -exec rm {} +    # batches files (faster)
```

Use `-exec` when you're already using `find`. Use `xargs` when you're combining multiple commands or need parallel processing.

The key insight: **pipes pass data between commands, but most commands need arguments, not data**. `xargs` is the bridge.

The beauty is these primitives compose infinitely. You can build complex data pipelines entirely from the command line:

```bash
cat data.json | jq '.items[]' | grep "active" | wc -l
```

This processes JSON with `jq` (a JSON processor that can query and transform), filters it with `grep`, searches, and counts—all in one line without any intermediate files or scripts. (We'll cover more advanced composition patterns, including how to handle commands that need arguments instead of stdin, in later sections.)

## Getting Help

Before you can become self-sufficient, you need to know how to find answers.

### Built-in Help

Most commands support `--help` or `-h`:

```bash
npm --help
grep --help
curl -h
```

This is usually the quickest way to see available options and basic usage.

### Man Pages

The traditional Unix documentation system:

```bash
man grep        # opens the manual page
man ls
man man         # yes, really

# sections: 1=commands, 2=syscalls, 3=library, 5=file formats, 8=admin
man 5 crontab   # file format vs man 1 crontab (command)
```

Navigate with arrow keys, `/` to search, `q` to quit. Man pages can be dense but they're comprehensive.

### Quick Lookups

Quick ways to identify and locate commands:

```bash
whatis grep     # one-line description
apropos search  # search man page descriptions (find commands)
which npm       # where the command lives
type cd         # shows if it's a builtin, alias, or external command

# for shell builtins
help cd
help export
```

### Info Pages

Some GNU tools use `info` instead of/alongside man:

```bash
info bash       # more detailed than man bash
info coreutils  # covers ls, cp, mv, etc.
```

### Command-Specific Patterns

Different tools have their own help conventions:

```bash
# many tools show usage when called incorrectly
grep            # shows usage/options

# version often reveals the tool
npm --version
node -v

# some tools have subcommand help
git help commit
git commit --help
npm help install

# docker/kubectl style
docker --help
docker run --help
```

### Community Help (tldr)

`tldr` is a community-maintained alternative to man pages with practical examples. It's not built-in, so you'll need to install it first:

```bash
# macOS (using Homebrew package manager)
brew install tldr

# Linux (using apt package manager on Debian/Ubuntu)
sudo apt install tldr

# or via npm (if you have Node.js)
npm install -g tldr
```

Once installed, it gives concise, example-focused help:

```bash
tldr tar        # shows common tar use cases
tldr find       # practical find examples
```

This is often more useful than man pages when you just want to see "how do I actually use this?"

**A note on package managers:** Commands like `brew` (Homebrew for macOS) and `apt` (Debian/Ubuntu Linux) are package managers - they install software from curated repositories. macOS doesn't have one built-in, so most developers install Homebrew (see [brew.sh](https://brew.sh)). Linux distributions come with their own: `apt` (Debian/Ubuntu), `yum` (RedHat/CentOS), `dnf` (Fedora), `pacman` (Arch). Think of them like `npm` but for system-level tools.

The trick is knowing the hierarchy: try `--help` first (fast, focused), then `man` for details, then `info` for GNU tools, then Google when you need examples or explanations. And `apropos` is underrated when you know *what* you want to do but not *which* command does it.

## Real-World Patterns

Theory is great, but here are the patterns you'll actually use and the gotchas you'll encounter.

### Quoting Rules and Gotchas

Quoting matters hugely. Without quotes, the shell splits on whitespace and expands wildcards:

```bash
file="my document.txt"
cat $file        # tries to cat "my" and "document.txt"
cat "$file"      # correct

rm *.txt         # expands before rm sees it
rm "*.txt"       # tries to delete a file literally named *.txt
```

### The find-exec Pattern

The `-exec` flag lets you run a command on every file that `find` discovers. Here's how it works:

```bash
find <where> <conditions> -exec <command> {} \;
#                                ^       ^  ^^
#                                |       |  |└─ marks end of command (semicolon must be escaped)
#                                |       |  └── OR use + to batch multiple files
#                                |       └───── placeholder: replaced with each found file path
#                                └─────────── the command to run
```

**The two terminators:**
- `\;` runs the command **once per file**: `rm file1.txt`, `rm file2.txt`, `rm file3.txt`
- `+` **batches files together**: `rm file1.txt file2.txt file3.txt` (much faster!)

**Simple examples:**

```bash
# delete all .tmp files (once per file)
find . -name "*.tmp" -exec rm {} \;

# delete all .tmp files (batched - faster!)
find . -name "*.tmp" -exec rm {} +

# find all .js files and search for "TODO" in each
find . -name "*.js" -exec grep "TODO" {} \;

# find files and show their size
find . -name "*.log" -exec ls -lh {} \;
```

This is safer than piping to xargs for files with weird names (spaces, quotes, etc.), though xargs with `-0` is also safe.

#### Real-World Example: Deleting node_modules Directories

This is a classic problem - you have multiple projects and want to clean up all `node_modules` directories. Let's compare approaches:

**The wrong way (dangerous):**

```bash
rm -rf **/node_modules
# Problems:
# - The shell expands ** which can hit command-line length limits
# - If expansion is huge, this can fail or behave unexpectedly
# - Less control over what gets deleted
```

**The right way:**

```bash
find . -name "node_modules" -type d -prune -exec rm -rf {} +
```

Let's break this down:
- `find .` - start searching from current directory
- `-name "node_modules"` - find things named node_modules
- `-type d` - only directories (not files)
- `-prune` - **don't descend into matched directories** (this is key! Once we find node_modules, don't look inside it)
- `-exec rm -rf {} +` - execute rm on batches of results
  - `{}` is replaced with the found paths
  - `+` means "batch multiple arguments" (faster than `\;` which runs once per item)

**Why this is better:**
1. **No command-line length limits** - find processes results incrementally
2. **More control** - you can add conditions like `-mtime +30` (older than 30 days)
3. **Faster** - `-prune` stops find from descending into each node_modules
4. **Safer** - you can test first with `-print` instead of `-exec rm`

**Test before deleting:**

```bash
# See what would be deleted
find . -name "node_modules" -type d -prune -print

# Count how many
find . -name "node_modules" -type d -prune | wc -l

# Show disk space that will be freed
find . -name "node_modules" -type d -prune -exec du -sh {} +
```

This pattern works for any repetitive cleanup task - just change the name and type!


### Process Substitution Patterns

These are incredibly useful:

```bash
# compare remote and local
diff <(ssh server 'ls /path') <(ls /path)

# provide config without a file
some-tool --config <(echo "setting=value")
```

### Parallel Processing

Backgrounding needs care:

```bash
# naive parallel - fires everything at once
for file in *.txt; do process "$file" & done
wait  # wait for all to complete

# controlled parallel with xargs -P
find . -name "*.txt" | xargs -P 4 -I {} process {}  # 4 at a time
```

### History Expansions

These are shortcuts you can type directly in your terminal. The shell expands them **before** running the command:

```bash
!!           # repeat last command
# if you just typed: ls -la
# typing !! runs: ls -la again

sudo !!      # common pattern: run last command with sudo
# you tried: npm install -g typescript
# got: permission denied
# type: sudo !!
# runs: sudo npm install -g typescript

!$           # last argument from previous command
# you typed: ls /very/long/path/to/directory
# now type: cd !$
# runs: cd /very/long/path/to/directory

!*           # all arguments from previous command
!-2          # command from 2 lines ago
^old^new     # replace text in last command
# you typed: git commit -m "fix tpyo"
# type: ^tpyo^typo
# runs: git commit -m "fix typo"

# practical:
mkdir /long/path/to/directory
cd !$        # cd to that directory

npm install express
vim !$       # edit the same argument
```

**How it works:** When you type `!!` or `!$`, the shell replaces it with the actual command/argument before executing. You can press `Up arrow` to see the expanded command before running it.

### Stdin Behavior

Commands often behave differently when stdin comes from keyboard vs pipe/file:

```bash
grep "pattern"          # waits for keyboard input
grep "pattern" file.txt # reads from file
cat file.txt | grep "pattern"  # reads from pipe

# test if stdin is a terminal
[ -t 0 ] && echo "interactive" || echo "piped"
# [ ] is the test command (also written as 'test')
# -t 0 checks if file descriptor 0 (stdin) is a terminal
# && means "if that's true, then..."
# || means "otherwise..."
```

## Advanced Techniques

These techniques give you fine-grained control when you need it.

### Here Documents and Here Strings

**Here documents** (`<<`) let you pass multi-line input directly:

```bash
cat <<EOF
multiple lines
of input
EOF

# useful for templating:
ssh server 'cat > config.txt' <<EOF
setting1=value
setting2=value
EOF
```

**Here strings** (`<<<`) for single-line input:

```bash
grep "pattern" <<< "search this string"
bc <<< "2 + 2"  # bc is a calculator
```

### File Descriptors and Advanced Redirection

Beyond stdin (0), stdout (1), stderr (2), you can use arbitrary numbers:

```bash
# swap stdout and stderr
command 3>&1 1>&2 2>&3

# redirect to multiple places with tee
command | tee output.txt  # writes to file AND shows on screen (tee splits output)

# discard output entirely
command > /dev/null 2>&1  # common pattern to silence everything
```

### PATH and Command Resolution

When you type `npm`, the shell searches directories in `$PATH` in order:

```bash
echo $PATH   # see the search path
which npm    # shows which npm will run
type npm     # more detailed info

# run specific version bypassing PATH
/usr/local/bin/node script.js

# run from current directory (not in PATH)
./my-script.sh
```

### Signals

`Ctrl-C` doesn't just "stop" a program, it sends **SIGINT** (interrupt signal). Programs can catch this and clean up. Different signals:

```bash
Ctrl-C       # SIGINT (polite interrupt)
Ctrl-Z       # SIGTSTP (suspend)
Ctrl-\       # SIGQUIT (quit with core dump)

kill PID     # sends SIGTERM (polite termination request)
kill -9 PID  # sends SIGKILL (immediate termination, can't be caught)

# background processes ignore SIGINT
command &    # won't be killed by Ctrl-C
```

### Job Control

We covered [background processes](#background-processes) earlier with the `&` operator. Here's how to manage them once they're running:

```bash
jobs         # list background jobs
fg           # bring most recent job to foreground
fg %1        # bring job 1 to foreground
bg           # continue suspended job in background
Ctrl-Z       # suspend current foreground process
disown       # detach job so it survives shell exit
```

This is essential when you've accidentally started a long-running process in the foreground.

## Customization and Configuration

Make the shell work the way you want it to.

### Profile Files

Your shell reads configuration files when it starts, giving you a place to customize your environment. This is where you can:
- Set up aliases and functions
- Modify your PATH to include custom directories
- Set environment variables (like your editor preference)
- Configure your prompt
- Load tools and frameworks

Different programs also add to these files - when you install Node.js, nvm, or other tools, they often add setup code to your profile. Understanding these files helps you maintain a clean, organized shell configuration.

When you open a terminal, bash reads configuration files in order:

**Login shells** (when you first log in):

```bash
/etc/profile        # system-wide
~/.bash_profile     # or ~/.bash_login or ~/.profile (first one found)
```

**Interactive shells** (new terminal windows):

```bash
~/.bashrc           # most customization goes here
```

**On exit:**

```bash
~/.bash_logout
```

Common pattern: make `~/.bash_profile` source `~/.bashrc` so everything loads consistently:

```bash
# in ~/.bash_profile
if [ -f ~/.bashrc ]; then
  source ~/.bashrc  # 'source' runs commands from the file in the current shell
fi
```

### Aliases

Shortcuts for commands - create short names for commands you type frequently:

```bash
alias ll='ls -la'       # now you can just type 'll' instead of 'ls -la'
alias gs='git status'   # type 'gs' for git status
alias ..='cd ..'
alias serve='python -m http.server'

# with arguments - need a function instead:
alias gco='git checkout'  # works fine
alias gcb='git checkout -b'  # works fine if always same args
```

### Functions

For anything needing logic or arguments:

```bash
# create and cd into directory
mkcd() {
  mkdir -p "$1" && cd "$1"
}

# find and kill process by name
killport() {
  lsof -ti:$1 | xargs kill -9  # lsof lists open files/ports
}

# quick git commit
gcm() {
  git commit -m "$*"
}
```

### PATH Modifications

Adding directories to your PATH so commands can be found:

```bash
# add to PATH
export PATH="$HOME/bin:$PATH"           # prepend (takes priority)
export PATH="$PATH:/usr/local/bin"      # append

# node/npm
export PATH="$HOME/.npm-global/bin:$PATH"

# common to check first:
if [ -d "$HOME/bin" ]; then
  export PATH="$HOME/bin:$PATH"
fi
```

### Environment Variables

Setting variables that affect program behavior:

```bash
export EDITOR=vim
export NODE_ENV=development
export HISTSIZE=10000           # command history length
export HISTFILESIZE=20000

# prompt customization
export PS1='\u@\h:\w\$ '       # user@host:path$
```

### Conditionals for Cross-Platform

Adapting your configuration for different operating systems:

```bash
# different behavior on macOS vs Linux
if [[ "$OSTYPE" == "darwin"* ]]; then
  alias ls='ls -G'              # macOS color flag
else
  alias ls='ls --color=auto'    # Linux color flag
fi
```

## Simple Scripting

So far we've been typing commands interactively - one at a time. But what if you need to run the same sequence of commands repeatedly? Or automate a complex task? That's where shell scripts come in.

A shell script is just a text file containing commands. Instead of typing them one by one, you save them in a file and run them all at once. Scripts can include all the patterns we've covered - pipes, conditionals, loops, variables - making them powerful automation tools.

Let's move from interactive commands to reusable automation.

### Shebang and Execution

Every script needs to declare its interpreter:

```bash
#!/bin/bash
# first line tells system which interpreter to use

echo "Hello, World!"
```

Make executable: `chmod +x script.sh`, then run: `./script.sh`

### Variables

Storing and using values in scripts:

```bash
name="Dave"
count=42

echo $name              # Dave
echo "$name"            # Dave (preferred - handles spaces)
echo "${name}_suffix"   # Dave_suffix (braces when needed)

# command output into variable
files=$(ls *.txt)
today=$(date +%Y-%m-%d)
```

### Arguments

Accessing values passed to your script:

```bash
# $1, $2, etc. are positional arguments
# $0 is the script name
# $# is argument count
# $@ is all arguments
# $* is all arguments as single string

echo "Script: $0"
echo "First arg: $1"
echo "All args: $@"
echo "Count: $#"

# check if argument provided
if [ -z "$1" ]; then
  echo "Usage: $0 <filename>"
  exit 1
fi
```

> **For complex argument parsing:** These built-in variables work for simple scripts, but for complex CLIs with flags and options, you might want a library. In Node.js, [yargs](https://yargs.js.org/) is popular for parsing command-line arguments - it builds on these same concepts but adds features like `--flag` parsing, help text, and validation.

### Conditionals

Making decisions in scripts:

```bash
# if statement
if [ -f "file.txt" ]; then
  echo "File exists"
elif [ -d "directory" ]; then
  echo "Directory exists"
else
  echo "Neither exists"
fi

# common tests
[ -f file ]     # file exists
[ -d dir ]      # directory exists
[ -z "$var" ]   # string is empty
[ -n "$var" ]   # string is not empty
[ "$a" = "$b" ] # strings equal
[ "$a" -eq "$b" ] # numbers equal
[ "$a" -lt "$b" ] # less than

# modern bash prefers [[  ]]
if [[ -f file.txt && -r file.txt ]]; then
  echo "File exists and is readable"
fi
```

### Loops

Repeating actions over collections or ranges:

```bash
# for loop over files
for file in *.txt; do
  echo "Processing $file"
  # do something with $file
done

# for loop over numbers
for i in {1..5}; do
  echo "Number $i"
done

# for loop over lines
while IFS= read -r line; do
  echo "Line: $line"
done < file.txt

# C-style loop
for ((i=0; i<5; i++)); do
  echo $i
done

# iterate over arguments
for arg in "$@"; do
  echo "Arg: $arg"
done
```

### Functions in Scripts

Organizing reusable logic:

```bash
#!/bin/bash

# function definition
process_file() {
  local file=$1  # local variable
  if [ ! -f "$file" ]; then
    echo "Error: $file not found"
    return 1
  fi

  # do something
  wc -l "$file"
}

# call it
process_file "data.txt"
```

### Error Handling

Making scripts fail safely:

```bash
#!/bin/bash
set -e  # exit on any error
set -u  # exit on undefined variable
set -o pipefail  # exit on pipe failure

# or combined:
set -euo pipefail
```

### Check Dependencies

Verifying required tools are installed before running your script:

```bash
# command -v checks if a command exists
# &> /dev/null silences all output
# ! negates the result
# so this means: "if node command does NOT exist"
if ! command -v node &> /dev/null; then
  echo "node not found"
  exit 1  # exit with error code
fi

# if we get here, node exists and the script continues
```

### Temp Files

Creating temporary files that clean up automatically:

```bash
tmpfile=$(mktemp)  # creates a unique temp file
trap "rm -f $tmpfile" EXIT  # 'trap' runs the command when script exits (even on error!)

# use $tmpfile safely
echo "data" > $tmpfile
process $tmpfile

# when script exits, the trap automatically deletes the temp file
```

### Read User Input

Getting interactive input during script execution:

```bash
read -p "Continue? (y/n) " answer
if [[ "$answer" = "y" ]]; then
  echo "Continuing..."
  # do the work
elif [[ "$answer" = "n" ]]; then
  echo "Cancelled."
  exit 0
else
  echo "Invalid input. Please enter y or n."
  exit 1
fi
```

### Simple Menu

Creating interactive menu selections:

```bash
select option in "Option 1" "Option 2" "Quit"; do
  case $option in
    "Option 1") echo "You chose 1"; break;;
    "Option 2") echo "You chose 2"; break;;
    "Quit") exit;;
  esac
done
```

### Handling Command Arguments

Here's a practical example combining arguments and menus:

```bash
#!/bin/bash

# function that takes an action argument
perform_action() {
  local action=$1
  
  case $action in
    start)
      echo "Starting service..."
      ;;
    stop)
      echo "Stopping service..."
      ;;
    restart)
      echo "Restarting service..."
      ;;
    *)
      echo "Usage: $0 {start|stop|restart}"
      exit 1
      ;;
  esac
}

# if argument provided, use it
if [ -n "$1" ]; then
  perform_action "$1"
else
  # no argument - show interactive menu
  echo "Select an action:"
  select action in "Start" "Stop" "Restart" "Quit"; do
    case $action in
      "Quit") exit;;
      *) perform_action "${action,,}"; break;;  # ${action,,} converts to lowercase
    esac
  done
fi
```

Now you can run it as `./script.sh start` or just `./script.sh` for a menu.

The key difference: **profile commands** configure your interactive shell experience (aliases, PATH, prompt), while **scripts** are small programs that automate tasks. Profile loads once when shell starts; scripts run on demand and exit.

## Users, Permissions, and Security

Unix was designed as a multi-user system from the start. Multiple people can use the same computer, each with their own files, and the system needs to prevent users from accidentally (or maliciously) interfering with each other's work or critical system files.

### Why Permissions Exist

Permissions solve three problems:

1. **Privacy** - Your files shouldn't be readable by other users
2. **Stability** - Regular users shouldn't be able to break the system
3. **Security** - Malicious programs running as your user can't damage system files

This is why you can't just `rm -rf /usr/bin` - you don't have permission, and that's a feature, not a bug.

### Users and Groups

Every file is owned by a user and a group:

```bash
ls -l
# -rw-r--r--  1 dave staff  1234 Dec 18 10:00 file.txt
#               ↑    ↑
#               user group
```

**Users** are individual accounts. Your username (see it with `whoami`) owns your home directory and files you create.

**Groups** are collections of users. On macOS, your primary group is usually `staff`. On Linux, it's typically your username. Groups let multiple users collaborate on files.

Common groups:
- `wheel` or `sudo` - users who can run commands as root
- `docker` - users who can run Docker commands
- `www-data` - web server user

View your groups: `groups` or `id -Gn`

### The Root User

The `root` user (UID 0) is the superuser - it can do anything. No permissions checks apply. This is powerful and dangerous.

On most modern systems, the root account is disabled. Instead, you use `sudo` (superuser do) to run individual commands as root.

### Understanding Permissions

Every file has three sets of permissions:

```bash
ls -l file.txt
# -rw-r--r--  1 dave staff  1234 Dec 18 10:00 file.txt
# ↑ rwx rwx rwx
# │  │   │   └─ others (everyone else)
# │  │   └───── group (staff)
# │  └───────── owner (dave)
# └──────────── type (- = file, d = directory, l = link)
```

**Permission types:**

```bash
r = read (4)     # view file contents / list directory
w = write (2)    # modify file / create files in directory
x = execute (1)  # run file as program / enter directory
```

**Reading permissions:**

```bash
-rw-r--r--
# owner: rw- (read, write)
# group: r-- (read only)
# others: r-- (read only)

drwxr-xr-x
# owner: rwx (read, write, execute/enter)
# group: r-x (read, execute/enter)
# others: r-x (read, execute/enter)
```

### Changing Permissions

Modify who can read, write, or execute files:

```bash
# symbolic mode (readable)
chmod +x script.sh          # add execute for everyone
chmod u+x script.sh         # add execute for user (owner)
chmod g+w file.txt          # add write for group
chmod o-r file.txt          # remove read for others
chmod go-rwx private.txt    # remove all permissions for group and others

# numeric mode (faster when you know it)
chmod 755 script.sh         # rwxr-xr-x
chmod 644 file.txt          # rw-r--r--
chmod 600 private.txt       # rw------- (owner only)
chmod 700 bin/              # rwx------ (owner only, common for ~/bin)

# numeric breakdown:
# 7 = 4+2+1 = rwx
# 6 = 4+2   = rw-
# 5 = 4+1   = r-x
# 4 = 4     = r--
# 0 = 0     = ---
```

### Changing Ownership

Transfer file ownership between users and groups:

```bash
# change owner (requires sudo for files you don't own)
sudo chown dave file.txt

# change group (you must be in the target group)
chown :staff file.txt

# change both
sudo chown dave:staff file.txt

# recursive
sudo chown -R dave:staff directory/
```

### Using sudo

`sudo` runs a command as root. You'll be prompted for your password (your password, not root's).

```bash
# install software (modifies /usr/local or /usr)
sudo npm install -g typescript

# edit system files
sudo vim /etc/hosts

# restart system services
sudo systemctl restart nginx

# view your sudo permissions
sudo -l
```

**When to use sudo:**

✅ Installing system-wide software
✅ Editing system configuration files
✅ Managing system services
✅ Modifying files outside your home directory

**When NOT to use sudo:**

❌ Never use `sudo npm install` in your project (it's a trap - makes files root-owned)
❌ Don't use `sudo` to fix permission errors in your home directory
❌ Avoid `sudo chmod -R 777` (this is almost always wrong)

If you get a permission error:
1. Ask: "Should I have permission to do this?"
2. If it's your file in your home directory, fix ownership: `sudo chown -R $USER:$USER ~/path`
3. If it's a system file, maybe you do need sudo
4. If it's in node_modules, you probably installed something with sudo by mistake

### Common Permission Patterns

Typical permission settings for different use cases:

```bash
# executable script (yours to run)
chmod +x deploy.sh

# private file (SSH keys, credentials)
chmod 600 ~/.ssh/id_rsa

# public file (web content, everyone can read)
chmod 644 index.html

# shared directory (group can write)
chmod 775 shared/

# your bin directory (only you can modify)
chmod 700 ~/bin/

# publicly readable directory
chmod 755 public/
```

### Fixing Common Permission Problems

**Problem:** `npm install` creates root-owned files

```bash
# fix node_modules ownership
sudo chown -R $USER:$USER node_modules/

# prevent it: never use sudo with npm in projects
# use nvm or manage node installation properly
```

**Problem:** Can't write to a directory

```bash
# check current permissions
ls -ld directory/
# drwxr-xr-x  2 dave staff  64 Dec 18 10:00 directory/
#               ^^^^
#               if this is your username, you own it

# if you own it, add write permission
chmod u+w directory/

# if you don't own it, check if you should
ls -l
sudo chown $USER directory/  # if appropriate
```

**Problem:** Script won't execute

```bash
# check if executable
ls -l script.sh

# add execute permission
chmod +x script.sh
```

### The Golden Rule

**Permissions exist to protect you.** If you're fighting permissions, stop and ask:
- "Why doesn't this work?"
- "What is the system protecting?"
- "Is there a better way to do this?"

Reaching for `sudo chmod 777` is like responding to a locked door by tearing down the wall. Sometimes the door is locked for a reason.

## Shells and Shell Frameworks

### What is a Shell, Really?

Everything we've covered so far has been about using bash—but bash is just one of many shells. A shell is the program that interprets your commands and talks to the operating system. It's the layer between you and the **kernel** (the core of the operating system that manages hardware, memory, and processes).

Different shells exist because they make different trade-offs: compatibility vs features, speed vs convenience, POSIX compliance vs user-friendliness. Some have been around since the 1970s; others are modern rewrites focused on better defaults and developer experience.

> **What is POSIX?** POSIX (Portable Operating System Interface) is a standard that defines how Unix-like systems should work. If a shell is "POSIX compliant," it means scripts written for it should work on any Unix system. This is why many scripts start with `#!/bin/sh` instead of `#!/bin/bash` - for maximum portability.

> **What is GNU?** GNU (GNU's Not Unix) is a project that created free, open-source versions of Unix tools. When we say "GNU extensions," we mean bash includes features beyond the basic POSIX standard - things that make it more powerful but might not work in other shells.

### The Main Shells

#### bash (Bourne Again Shell)

**The default almost everywhere** (or it was until recently):

- Been around since 1989
- POSIX compliant with GNU extensions
- What most Linux distributions use
- What macOS used until Catalina (2019)
- Massive ecosystem of existing scripts
- If you learn bash, you can use most Unix systems

**Strengths:** Universal, stable, huge community, massive amount of existing documentation and scripts.

**Weaknesses:** Showing its age, awkward syntax for modern programming patterns, limited interactive features out of the box.

#### zsh (Z Shell)

**The modern default** (now default on macOS):

- Started in 1990, modernized over time
- Compatible with bash scripts (mostly)
- Better interactive features: better tab completion, spelling correction, shared history across sessions
- More powerful customization
- Supports plugins and themes

**Strengths:** Modern features while maintaining compatibility, excellent tab completion, better defaults, huge plugin ecosystem (thanks to [oh-my-zsh](https://ohmyz.sh/)).

**Weaknesses:** Slightly slower than bash, more complex configuration, not always installed by default on Linux.

**When to use it:** If you want a better interactive experience without leaving bash compatibility behind. If you're on macOS, you're already using it.

```bash
# check if you're using zsh
echo $SHELL
# /bin/zsh or /usr/bin/zsh

# switch to zsh
chsh -s $(which zsh)
# log out and back in
```

#### fish (Friendly Interactive Shell)

**The radical rethink:** ([fishshell.com](https://fishshell.com/))

- Started in 2005
- NOT bash-compatible—different syntax entirely
- Incredible defaults: syntax highlighting, autosuggestions, fantastic tab completion
- Web-based configuration
- Simpler, more consistent scripting syntax
- No configuration needed to be useful

**Strengths:** Best out-of-the-box experience, beautiful by default, intuitive scripting, great documentation.

**Weaknesses:** Not bash-compatible (your bash scripts won't work), smaller ecosystem, not installed by default anywhere, has to translate environment variables from bash profiles.

**When to use it:** If you're willing to learn different syntax for a dramatically better interactive experience. If you value productivity over compatibility. If you hate configuring things.

```fish
# fish has a completely different syntax
set name "Dave"  # not name="Dave"
if test -f file.txt  # not [ -f file.txt ]
    echo "exists"
end  # not fi

# but the experience is amazing
# type any command and get:
# - syntax highlighting in real-time
# - command suggestions based on history
# - incredible tab completion
```

#### Others Worth Knowing About

**sh (Bourne Shell)** - The original, POSIX standard. On modern systems, `/bin/sh` is usually a symlink to bash or dash. Used in scripts for maximum portability.

**dash (Debian Almquist Shell)** - Minimal, fast, POSIX-compliant. Often `/bin/sh` on Debian/Ubuntu. Used for system scripts where speed matters.

**ksh (Korn Shell)** - Popular in enterprise Unix. Influenced bash heavily.

### Shell Frameworks and Enhancement Tools

Once you've picked a shell, you can add frameworks and tools to make it even better.

#### oh-my-zsh

**The most popular zsh framework:**

- 300+ plugins (git, docker, node, npm, etc.)
- 150+ themes
- Convention-based configuration
- Huge community

```bash
# install (check website for latest command at ohmyz.sh)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
# this downloads and runs an installer script - a common pattern for dev tools

# edit ~/.zshrc
plugins=(git node npm docker kubectl)
ZSH_THEME="robbyrussell"
```

**Pros:** Everything you need in one place, well-maintained, extensive documentation.

**Cons:** Can be slow with too many plugins, sometimes bloated, opinionated structure.

**When to use it:** You want zsh superpowers without building your own config from scratch. You value convenience over performance.

#### Starship

**The cross-shell prompt:** ([starship.rs](https://starship.rs/))

- Works with bash, zsh, fish, and others
- Written in Rust (fast)
- Beautiful default prompt
- Shows git status, language versions, kubernetes context, etc.
- Highly customizable

```bash
# install
brew install starship  # macOS (Homebrew package manager)
# or
curl -sS https://starship.rs/install.sh | sh  # universal installer

# add to ~/.bashrc or ~/.zshrc
eval "$(starship init bash)"  # or zsh, fish, etc.
# 'eval' executes the output of the command as shell code
```

**Pros:** Fast, gorgeous, cross-shell, single config file, minimal setup.

**Cons:** Limited to prompt customization (doesn't provide plugins or aliases).

**When to use it:** You want a beautiful, informative prompt without committing to a full framework. You switch between shells.

#### Powerlevel10k

**The fastest, most feature-rich zsh theme:** ([github.com/romkatv/powerlevel10k](https://github.com/romkatv/powerlevel10k))

- Extremely fast (even with complex prompts)
- Configuration wizard walks you through setup
- Shows everything: git, languages, time, command duration
- Instant prompt (shows prompt before zsh fully loads)

```bash
# install (with oh-my-zsh)
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
# downloads the theme from GitHub into your oh-my-zsh themes directory

# set in ~/.zshrc
ZSH_THEME="powerlevel10k/powerlevel10k"

# run configuration wizard
p10k configure
```

**Pros:** Blazingly fast, incredible configurability, beautiful, well-maintained.

**Cons:** zsh only, can be overwhelming with all the options.

**When to use it:** You want the best zsh prompt and don't mind some complexity.

#### prezto

**The lighter oh-my-zsh alternative:**

- Leaner, faster than oh-my-zsh
- Better default configurations
- Modular design
- Less plugins, but more curated

**When to use it:** You want zsh enhancements but oh-my-zsh feels too heavy.

#### zinit (formerly zplugin)

**The zsh plugin manager:**

- Super fast (turbo mode for lazy loading)
- Fine-grained control over plugin loading
- Works with oh-my-zsh plugins
- Steeper learning curve

**When to use it:** You want maximum performance and control. You're comfortable with configuration.

#### fish-specific frameworks

**fisher** - Minimal fish plugin manager

**oh-my-fish** - The oh-my-zsh equivalent for fish (but less needed due to fish's great defaults)

### Making the Choice

Here's how I'd think about it:

#### Start with your system default

**On macOS:** You have zsh. Add Starship for a better prompt, then gradually add plugins as you need them.

**On Linux:** You probably have bash. Try it for a while. If you want more features, install zsh and oh-my-zsh.

#### For the best interactive experience immediately

Install **fish**. Seriously, just try it. It's different but the experience is night-and-day better for interactive use. Keep bash around for running scripts.

```bash
# macOS (Homebrew package manager)
brew install fish

# Linux (apt package manager on Debian/Ubuntu)
sudo apt install fish

# then try it
fish
# play around
# if you like it:
chsh -s $(which fish)
```

#### For bash compatibility with modern features

Install **zsh** + **oh-my-zsh** + **powerlevel10k**. This is the "safe" upgrade path—most of your bash knowledge transfers, but you get autocompletion, syntax highlighting, git integration, and more.

#### For maximum performance

Use **zsh** + **zinit** + **Starship**. Lazy-load plugins, optimize everything. Or just use **bash** with **Starship**—bash is fast, and Starship is fast.

#### For minimal configuration

Install **fish**. It's good out of the box. Or use **bash** + **Starship** for a quick win.

### Migration Tips

#### Switching shells doesn't lose your work

Your aliases, functions, and scripts are just text files. Moving from bash to zsh:

```bash
# copy your bash config to zsh
cat ~/.bashrc >> ~/.zshrc
# then edit and adapt as needed
```

Fish is different, but you can translate:

```bash
# bash
export PATH="$HOME/bin:$PATH"
alias ll='ls -la'

# fish
set -gx PATH $HOME/bin $PATH
alias ll='ls -la'  # same!
```

#### You can use multiple shells

```bash
# default shell is zsh, but run bash for a script
bash my-script.sh

# start a fish session temporarily
fish
# play around
exit  # back to zsh

# run a one-off command in bash
bash -c 'echo $BASH_VERSION'
```

#### Test before committing

Don't `chsh` immediately. Just type `zsh` or `fish` in your current shell to try it out. When you're ready:

```bash
chsh -s $(which zsh)  # or fish
# log out and back in
```

### The Reality Check

Honestly? For most developers:

- **bash + Starship** is perfectly fine and universal
- **zsh + oh-my-zsh** is the sweet spot of features vs familiarity
- **fish** is the best experience if you're willing to learn different syntax

The shell doesn't make you a better developer. But a nice prompt, good tab completion, and some convenient aliases can make your day-to-day work more pleasant. Start simple, add complexity only when you need it.

And remember: all the command line fundamentals we covered earlier work in all of these shells. Pipes, redirection, file operations—those are universal. The shell is just the interpreter.

## Where to Go From Here

If you've made it this far, you've got a solid foundation. You understand how the shell processes commands, how the filesystem is organized, how to chain tools together, and how to customize your environment. That's honestly 90% of what you'll use day-to-day.

The remaining 10% comes from experience: discovering that `xargs` can solve a problem you've been struggling with, learning `sed` and `awk` when you need to transform text, figuring out `rsync` for deployments, or diving into shell scripting when you need more complex automation.

### Some Practical Next Steps

**Start customizing.** Add that alias you keep wishing existed. Write a function to automate something you do multiple times a week. The best way to learn is to scratch your own itches.

**Read others' dotfiles.** GitHub is full of people's bashrc and zshrc configurations. You'll discover patterns and tools you never knew existed. (Don't cargo-cult entire configs, but steal the good bits.)

**Learn one power tool deeply.** Pick `find`, `grep`, `awk` (text processing by columns), `sed` (stream editing/find-replace), or `jq` (for JSON) and really understand it. Having one tool you're genuinely fluent in opens up a lot of possibilities.

**Practice composing pipelines.** Next time you need to process some data, resist the urge to write a Node script. See if you can solve it with a pipeline of standard tools. Sometimes a script is the right answer, but often `cat data.json | jq '.items[]' | grep "active" | wc -l` is faster and more maintainable.

The command line isn't going anywhere. The tools we covered—pipes, redirection, the filesystem hierarchy—haven't fundamentally changed in 40+ years. Time invested learning these patterns pays dividends across your entire career, regardless of what languages or frameworks you're using next year.

Now go forth and `rm -rf` with confidence. (But maybe double-check that path first.)

## Quick Reference

### Filesystem Navigation

```bash
pwd                 # print working directory
cd path             # change directory
cd                  # go home
cd -                # go to previous directory
ls -la              # list all files with details
tree -L 2           # show directory tree 2 levels deep
```

### File Operations

```bash
touch file          # create/update file
mkdir -p path/to/dir # create directory with parents
cp -r src dest      # copy recursively
mv src dest         # move/rename
rm -rf dir          # delete recursively (careful!)
ln -s target link   # create symlink
```

### Finding Things

```bash
find . -name "*.js" # find files by name
find . -type f      # find files only
which command       # locate command in PATH
grep -r "text" .    # search text recursively
du -sh *            # disk usage summary
df -h               # disk free space
```

### Redirection & Pipes

```bash
cmd > file          # redirect stdout to file
cmd >> file         # append stdout to file
cmd 2> file         # redirect stderr to file
cmd &> file         # redirect both to file
cmd1 | cmd2         # pipe stdout to next command
cmd < file          # read stdin from file
```

### Command Chaining

```bash
cmd1; cmd2          # run sequentially
cmd1 && cmd2        # run cmd2 if cmd1 succeeds
cmd1 || cmd2        # run cmd2 if cmd1 fails
cmd &               # run in background
```

### History & Shortcuts

```bash
!!                  # repeat last command
!$                  # last argument of previous command
!*                  # all arguments of previous command
^old^new            # replace in last command
Ctrl-R              # reverse search history
```

### Variables & Environment

```bash
VAR=value           # set variable
export VAR=value    # set environment variable
echo $VAR           # use variable
echo $PATH          # view PATH
export PATH="$HOME/bin:$PATH"  # prepend to PATH
```

### Permissions & Security

```bash
# view permissions
ls -l file          # see ownership and permissions
whoami              # current user
groups              # your groups
id                  # detailed user info

# change permissions
chmod +x script     # add execute
chmod 755 file      # rwxr-xr-x (owner all, others read+execute)
chmod 644 file      # rw-r--r-- (owner write, others read)
chmod 600 file      # rw------- (owner only, for private files)

# change ownership
sudo chown user file         # change owner
sudo chown user:group file   # change owner and group
sudo chown -R user dir/      # recursive

# run as superuser
sudo command        # run single command as root
sudo -i             # interactive root shell (avoid)
sudo !!             # run last command with sudo
```

### Common Patterns

```bash
# find and delete
find . -name "*.tmp" -delete

# find and execute
find . -name "*.js" -exec grep "TODO" {} \;

# xargs: convert stdin to arguments
find . -name "*.log" | xargs rm
ls *.txt | xargs -I {} cp {} {}.backup

# process substitution
diff <(cmd1) <(cmd2)

# command substitution
echo "Today is $(date)"

# background and wait
cmd1 & cmd2 & cmd3 &
wait

# conditional execution
[ -f file ] && echo "exists"
```

### Common Commands

```bash
# text processing
grep "pattern" file     # search for pattern in file
grep -r "text" .        # recursive search
cat file                # display file contents
echo "text"             # print text
wc -l file              # count lines
sort file               # sort lines
uniq file               # remove duplicates

# data transformation
cut -d',' -f1 file      # extract column from CSV
sed 's/old/new/g' file  # find and replace
awk '{print $1}' file   # process columns
jq '.key' file.json     # query JSON

# building commands
xargs command           # build commands from stdin (converts lines to arguments)
xargs -I {} cmd {}      # use placeholder for each item
xargs -P 4 cmd          # run 4 in parallel

# output control
tee file                # write to file AND stdout
head -n 10 file         # first 10 lines
tail -n 10 file         # last 10 lines
tail -f file            # follow file (watch changes)
less file               # paginated viewer

# process management
ps aux                  # list all processes
top                     # interactive process viewer
kill PID                # terminate process
killall name            # kill by process name
lsof -ti:3000           # find process on port 3000

# system info
date                    # current date/time
uptime                  # system uptime
whoami                  # current user
hostname                # system name
uname -a                # system information
```

### Getting Help

```bash
man command         # manual page
command --help      # built-in help
which command       # locate command
type command        # command type (alias/builtin/file)
apropos keyword     # search man pages
```
