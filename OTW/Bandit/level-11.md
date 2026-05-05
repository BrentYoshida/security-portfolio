# Bandit Level 11

### Goal
The goal of this level is to find the password for the next level: which is stored in a file called `data.txt` where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

### Commands Used
* `ls`
* `cat`
* `tr`

### Walkthrough
1. **Locate the file:** I started by listing the contents of the home directory to confirm the location of `data.txt`.
   ```bash
   ls
   # Output: data.txt
   ```

2. **Inspect the content:** Reading the file directly shows the scrambled ROT13 text.
   
```bash
   cat data.txt
   # Output: Gur cnffjbeq vf WHR7p6aWbb87ptv7XURpS09vS7UURSYW
   ```

3. **Decode the string:** While online tools like rot13.com work: the challenge is designed to be solved within the CLI using the `tr` (translate) command. This command maps one set of characters to another. 
   
   To reverse a 13-character shift: I mapped the alphabet starting at 'A' to the alphabet starting at 'N' (and similarly for lowercase).
   
```bash
   cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
   # Output: The password is JRE7c6nJoo87cgi7KHEpF09iF7HHEFLY
   ```

### Lessons Learned

#### The tr Command
The `tr` utility is used for translating or deleting characters. It is a powerful tool for quick encoding or decoding tasks that involve character mapping.

* **Syntax:** `tr [SET1] [SET2]`
* **How it works:** It replaces each character in SET1 with the character at the same position in SET2.
* **Why it works for ROT13:** Since the alphabet has 26 letters: shifting by 13 is its own inverse. Mapping `A-M` to `N-Z` and `N-Z` to `A-M` perfectly swaps the characters back to their original state.



#### ROT13 (Rotate 13)
ROT13 is a simple substitution cipher that replaces a letter with the 13th letter after it in the alphabet. 

* It is a special case of the **Caesar Cipher**.
* It only affects alphabetic characters: numbers and symbols remain unchanged.
* It is often used in online forums to hide spoilers or offensive material: as it is easily reversible but not readable at a glance.

> **Pro Tip:** In a pinch: you can also use `alias` or simple Python one-liners to decode ROT13: but `tr` is the most "Linux-native" way to handle character translation in a pipeline.
```
