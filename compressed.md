````markdown
# File Compression and Archiving in Ubuntu Server

This guide explains how to manage file compression and archiving using the terminal in Ubuntu Server, focusing on the **bzip2**, **bunzip2**, and **tar** commands.

In Linux environments such as Ubuntu Server, file compression is important for:
- Reducing disk space usage
- Bundling multiple files or directories into a single archive
- Simplifying file transfer and backup operations

---

# Part 1: Basic File Compression (bzip2 and bunzip2)

The **bzip2** utility compresses individual files, while **bunzip2** decompresses them.

**Important:**  
These commands compress **single files only**. They do **not combine multiple files or directories**.

---

## 1. Compressing a File

To compress a file:

```bash
bzip2 psu1.txt
````

**Result:**

* `psu1.txt` is removed
* `psu1.txt.bz2` is created

### Keeping the Original File

To compress while keeping the original file, use the `-k` flag:

```bash
bzip2 -k psu2.txt
```

**Result:**

* `psu2.txt`
* `psu2.txt.bz2`

Both files remain.

---

## 2. Decompressing a File

To extract a `.bz2` file:

```bash
bunzip2 psu1.txt.bz2
```

**Result:**

* `psu1.txt` is restored
* `psu1.txt.bz2` is deleted

### Keeping the Compressed File

Use the `-k` flag:

```bash
bunzip2 -k psu3.txt.bz2
```

**Result:**

* Extracted file remains
* `.bz2` archive is preserved

---

# Part 2: Archiving and Compressing with tar

While **bzip2** works on single files, **tar (Tape Archive)** is used to combine multiple files and directories into one archive.

`tar` can also compress the archive using algorithms such as **gzip** or **bzip2**.

---

# Understanding Common tar Flags

| Flag | Meaning                                |
| ---- | -------------------------------------- |
| c    | Create a new archive                   |
| x    | Extract files from an archive          |
| t    | List contents of an archive            |
| f    | Specifies the archive filename         |
| v    | Verbose output (shows processed files) |
| z    | Use gzip compression (`.tar.gz`)       |
| j    | Use bzip2 compression (`.tar.bz2`)     |

**Note:**
The `f` flag must be the **last option before the archive filename**.

---

# Working with `.tar.gz` (Gzip Compression)

**Gzip** is generally faster but produces slightly larger archives than bzip2.

---

## 1. Create a `.tar.gz` Archive

```bash
tar cfzv data.tar.gz data
```

Explanation:

* `c` create archive
* `f` archive filename
* `z` gzip compression
* `v` verbose output

The folder `data` becomes `data.tar.gz`.

---

## 2. View Contents of a `.tar.gz` Archive

```bash
tar tfzv data.tar.gz
```

Lists files without extracting them.

---

## 3. Extract a `.tar.gz` Archive

```bash
tar xfzv data.tar.gz
```

Extracts the archive into the current directory.

---

# Working with `.tar.bz2` (Bzip2 Compression)

**Bzip2** compression is slower but typically produces **smaller archives**.

---

## 1. Create a `.tar.bz2` Archive

```bash
tar cfjv data.tar.bz2 data
```

Explanation:

* `j` uses bzip2 compression.

---

## 2. View Contents of a `.tar.bz2` Archive

For large archives:

```bash
tar tfjv httpd.tar.bz2 | more
```

This allows scrolling through the list of files.

---

## 3. Extract a `.tar.bz2` Archive

```bash
tar xfjv httpd.tar.bz2
```

Extracts the archive.

---

# Part 3: Common Pitfalls

Example commands:

```bash
tar xfjv data.tar.gz
tar xfzv data.tar.bz2
```

These commands produce **errors**.

### Why?

* The first command uses the **bzip2 flag (`j`)** on a **gzip archive (`.gz`)**.
* The second command uses the **gzip flag (`z`)** on a **bzip2 archive (`.bz2`)**.

### Correct Rule

Match the flag with the archive type:

| Archive Type          | Flag |
| --------------------- | ---- |
| `.tar.gz` or `.tgz`   | `z`  |
| `.tar.bz2` or `.tbz2` | `j`  |

---

# Modern tar Behavior

Recent Ubuntu versions can **auto-detect compression formats**.

Example:

```bash
tar xfv archive_name
```

`tar` may extract the archive correctly without specifying `z` or `j`.

However, explicitly specifying the correct flags remains **best practice** for system administration.

```
```
