# OverTheWire Bandit Notes

This game helped me learn the basics of **Linux** and **Git** commands

## What I learned in levels 0-10:

---

## SSH Connections

- Used SSH to connect to each level's machine.
- Learned the importance of the `-p` option to specify a port.
- Learned the `-i` option to specify a private key file for authentication.

---

## Understanding Standard Input, Output, and Special Characters

- Directly specifying a filename starting with `-` doesn't work because `-` refers to **Standard Input (stdin)** or **Standard Output (stdout)** in Linux.
- `stdin` (stream ID 0) is the default input source (usually keyboard).
- `stdout` (stream ID 1) is the default output destination (usually the screen).
- `stderr` (stream ID 2) is the stream for error messages.
- Since `-` is a special character (used for options or stdin/stdout), filenames starting with `-` must be referenced differently, e.g. by prepending `./` like `./-`.

---

## Handling Filenames with Spaces

- Files or directories with spaces in their names must be handled carefully.
- Without special treatment, commands interpret spaces as argument separators.
- Two ways to handle spaces:
  - Escape spaces with backslash: `spaces\ in\ filename`
  - Enclose the filename in quotes: `"spaces in filename"` or `'spaces in filename'`

---

## Best Practices for Filenames

- Avoid using spaces in filenames or directory names.
- Use underscores `_` or dashes `-` instead.
- Spaces can break scripts or commands because the shell splits arguments by spaces.

---

## Examples

- Viewing a file named `spaces in this filename`:

```bash
cat "spaces in this filename"
cat ./-filename
