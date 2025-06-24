# OverTheWire Bandit Notes

This game helped me learn the basics of **Linux** and **Git** commands.

## What I learned in levels 0-10:

---

## SSH Connections

Using SSH to connect to each level's machine is essential. The `-p` option specifies a port, while the `-i` option allows authentication with a private key file. Understanding these options makes connecting to remote systems more flexible and secure.

---

## Understanding Standard Input, Output, and Special Characters

Directly specifying a filename starting with `-` doesn't work because `-` refers to **Standard Input (stdin)** or **Standard Output (stdout)** in Linux.  
- `stdin` (stream ID 0) is the default input source (usually the keyboard).
- `stdout` (stream ID 1) is the default output destination (usually the screen).
- `stderr` (stream ID 2) is the stream for error messages.

Since `-` is a special character (used for options or stdin/stdout), filenames starting with `-` must be referenced differently, for example by prepending `./` like `./-filename`. This ensures the shell interprets it as a filename and not as an option.

---

## Handling Filenames with Spaces

Files or directories with spaces in their names must be handled carefully. Without special treatment, commands interpret spaces as argument separators.  
There are two ways to handle spaces:
- Escape spaces with a backslash: `spaces\ in\ filename`
- Enclose the filename in quotes: `"spaces in filename"` or `'spaces in filename'`

This prevents the shell from splitting the filename into separate arguments, ensuring the command works as intended.

---

## Best Practices for Filenames

Avoid using spaces in filenames or directory names. Using underscores `_` or dashes `-` instead is recommended. Spaces can break scripts or commands because the shell splits arguments by spaces. Consistent naming makes automation and scripting much easier and more reliable.

---

## Examples

Viewing a file named `spaces in this filename`:

```bash
cat "spaces in this filename"
cat ./-filename
```

---

## Detecting Human-Readable Files

The `file` command helps determine the type of data in a file, distinguishing between human-readable text (like ASCII or Unicode) and binary/executable files (like ELF).  
Checking a single file is straightforward:

```bash
file ./filename
```

To identify which files in a directory are human-readable (ASCII), use the `file` command on all files:

```bash
file ./*
```

To search recursively through all subdirectories and filter for ASCII files:

```bash
find . -type f -exec file {} \; | grep 'ASCII'
```

This approach quickly locates all ASCII (human-readable) files, even if they have unusual names or are hidden in subfolders. Using wildcards like `*` allows commands to be applied to multiple files at once, making batch operations efficient.

For even more reliable detection, MIME types can be used:

```bash
find . -type f -exec file --mime {} \; | grep 'text/'
```

If a file is not human-readable, trying to `cat` it may result in unreadable or garbage output (e.g., symbols like `����`).

---

## Searching for a File Matching Specific Conditions

To find a file that is human-readable (ASCII text), has an exact size of 1033 bytes, is not executable, and is located somewhere inside the `inhere/` directory, the following command is used:

```bash
find inhere -type f -size 1033c ! -executable -exec file {} \; | grep ASCII
```

This command searches for regular files in the `inhere/` directory that are exactly 1033 bytes, are not executable, and are confirmed as ASCII text by the `file` command. Filtering with `grep ASCII` ensures only human-readable files are listed.  
Combining these options allows for precise searching, which is especially useful when dealing with many files or when filenames are unknown or complex.

---

## Linux File Permissions and Ownership

Every file in Linux has an owner (user) and a group. The `ls -l` command displays this information, showing both the user and group for each file. Ownership is important for managing access and permissions. To search for files based on ownership and size, the `find` command is very effective. For example, searching for files owned by user `bandit7`, group `bandit6`, and exactly 33 bytes in size can be done with:

```bash
find / -type f -user bandit7 -group bandit6 -size 33c
```

This command searches the entire filesystem (`/`), looks only for files (`-type f`), filters by user and group ownership, and matches files that are exactly 33 bytes (`-size 33c`). This is useful for pinpointing files with very specific attributes, especially in CTF challenges or system administration tasks.

---

## Extracting Human-Readable Strings from Files

The `strings` command is used to extract sequences of printable characters from files, which is especially useful for binary or non-text files. For example, to find lines in `data.txt` that contain three equal signs (which might indicate a password or special value), you can use:

```bash
strings data.txt | grep ===
```

This filters the output of `strings` to show only lines with three equal signs. Adjusting the number of equal signs in the `grep` pattern can help narrow down the results, depending on the file's content.


---

## What I learned in levels 10-20:

## Working with Base64 Encoding and Decoding

Base64 is a common encoding scheme that converts binary data into ASCII text, often recognizable by equal signs at the end of the encoded string. The `base64` command in Linux allows for both encoding and decoding. Decoding is done with the `-d` flag:

```bash
base64 -d data.txt
```

Base64 encoding is not encryption; it simply transforms data for easier transmission or storage. The encoded output can be easily decoded back to its original form. The command can work with files or standard input/output, and the results can be redirected to files or displayed in the terminal. This is a fundamental tool for handling encoded data in Linux environments.

---


## Substitution Ciphers and ROT13 with `tr`

Substitution ciphers work by replacing each character in the plaintext with another character. Classic examples include the Caesar and Vigenère ciphers, but these are no longer considered secure. One of the simplest substitution ciphers is ROT13, which rotates each letter by 13 positions in the alphabet. Since the Latin alphabet has 26 letters, applying ROT13 twice returns the original text, making the encryption and decryption process identical.

The `tr` command in Linux is used for character translation or substitution. It can be used to implement ROT13 directly in the terminal. For ROT13, the substitution is as follows:  
- Letters A–M become N–Z  
- Letters N–Z become A–M  
- The same applies for lowercase letters a–m and n–z

To apply ROT13 to the contents of a file:

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

To decode ROT13 (which is the same operation):

```bash
echo 'hello world' | tr 'N-ZA-Mn-za-m' 'A-Za-z'
```

This approach allows for quick and easy letter substitution directly from the command line, without the need for external tools or websites. The `tr` command is a powerful utility for simple text transformations in Linux.