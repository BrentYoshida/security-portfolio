# Bandit Level 12

### Goal
The goal of this level is to find the password for the next level, which is hidden inside a file that has been repeatedly compressed and archived using various formats.

### Commands Used
* `mkdir`
* `cd`
* `cp`
* `ls`
* `file`
* `xxd`
* `bzcat`
* `zcat`
* `tar`
* `cat`

### Walkthrough
1. **Prepare a working directory:** Since the home directory might have restricted permissions, I created a temporary directory in `/tmp` to work in.
   ```bash
   cd /tmp
   mkdir testdir1
   cd testdir1
   ```

2. **Copy the data file:** I copied the `data.txt` file from the home directory into my new folder.
   ```bash
   cp ~/data.txt .
   ```

3. **Identify and extract the file:** The process required identifying the file type using `file` and then using the appropriate decompression or extraction command. I repeated this cycle until the password was revealed.

   * **Step 1:** Used `xxd -r` to convert a hex dump to binary, resulting in a gzip-compressed file.
     
```bash
     xxd -r data.txt > file1
     file file1
     # Result: gzip compressed data
     zcat file1 > file2
     ```
   * **Step 2:** Identified `file2` as a bzip2 archive.
     
```bash
     file file2
     # Result: bzip2 compressed data
     bzcat file2 > file3
     ```
   * **Step 3:** Identified `file3` as a gzip-compressed file.
     
```bash
     file file3
     # Result: gzip compressed data
     zcat file3 > file4
     ```
   * **Step 4:** Identified `file4` as a tar archive and extracted it.
     
```bash
     file file4
     # Result: POSIX tar archive
     tar -xvf file4
     # Result: Extracted data5.bin
     ```
   * **Step 5:** Continued this process of identifying and extracting `data5.bin`, `data6.bin`, and `data8.bin` until the final file was retrieved.
     
```bash
     file data5.bin
     # Result: POSIX tar archive
     tar -xvf data5.bin
     
     file data6.bin
     # Result: bzip2 compressed data
     bzcat data6.bin > file7
     
     file file7
     # Result: POSIX tar archive
     tar -xvf file7
     
     file data8.bin
     # Result: gzip compressed data
     zcat data8.bin > file9
     ```

4. **Retrieve password:**
   
```bash
   cat file9
   # Output: The password is FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn
   ```

---

### Lessons Learned

Choosing the right command in Bash depends entirely on the **file extension** and whether you need to **decompress** data on the fly. Using the wrong one usually results in a terminal screen full of gibberish.

#### 1. cat (Concatenate)
The standard workhorse. Use this for plain, uncompressed text files.
* **Use when:** The file is a standard .txt, .log, .sh, or has no extension.
* **What it does:** Reads the file and pipes the raw content to your terminal or another command.

#### 2. zcat (Gzip Cat)
Think of this as cat for files compressed with **gzip**.
* **Use when:** The file ends in **.gz**.
* **What it does:** Decompresses the file in memory and displays the text without you having to manually run gunzip.

#### 3. bzcat (Bzip2 Cat)
The equivalent of zcat, but specifically for the **bzip2** compression format.
* **Use when:** The file ends in **.bz2**.
* **What it does:** Decompresses .bz2 files to standard output.

#### 4. tar (Tape Archiver)
Unlike the others, tar isn't just for reading; it’s for **grouping** many files into one.
* **Use when:** You are dealing with **.tar**, **.tar.gz**, or **.tgz** files.
* **Common Flags:**
    * `tar -tf file.tar`: **List** contents (Table of contents).
    * `tar -xf file.tar`: **Extract** contents.

### Quick Reference Table

| Command | Target Extension | Primary Purpose |
|---|---|---|
| cat | .txt, .log, etc. | View plain text files. |
| zcat | .gz | View gzip-compressed text. |
| bzcat | .bz2 | View bzip2-compressed text. |
| tar | .tar, .tar.gz | Bundle or extract multiple files/folders. |

> **Pro Tip:** If you aren't sure what a file is, run `file <filename>` first. It will tell you if it's "ASCII text," "gzip compressed data," or a "POSIX tar archive" so you can pick the right tool for the job.
```
