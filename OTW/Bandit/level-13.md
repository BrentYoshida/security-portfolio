# Bandit Level 13

### Goal
The goal of this level is to log into `bandit14` using a private SSH key stored in the home directory. The password for the next level is not in a text file but is accessible only after successfully authenticating as the `bandit14` user.

### Commands Used
* `ls`
* `cat`
* `ssh`
* `chmod`
* `nano` (on local VM)

### Walkthrough
1. **Identify the key:** Upon logging in, I found a file named `sshkey.private` and a `HINT` file.
   ```bash
   ls
   # Output: HINT  sshkey.private
   ```

2. **Understand the obstacle:** The game blocks SSH connections from `localhost` to conserve resources. This meant I couldn't simply run `ssh -i sshkey.private bandit14@localhost` from within the `bandit13` session.

3. **Transfer the key to a local VM:** To bypass the localhost restriction, I needed to use the key from my own machine (Kali Linux). 
   * I used `cat sshkey.private` to display the RSA private key content.
   * I copied the entire block: including the `BEGIN` and `END` headers.
   * On my local Kali VM: I created a new file called `bandit14.key` using `nano` and pasted the content.

4. **Set strict permissions:** SSH will reject private keys that are "too open" (readable by other users). I had to modify the permissions so only my user could read the file.
   
```bash
   chmod 600 bandit14.key
   ```

5. **Authenticate as Bandit 14:** Using the `-i` (identity) flag: I pointed SSH to my newly created key file to log in directly to the OverTheWire server.
   
```bash
   ssh -i bandit14.key bandit14@bandit.labs.overthewire.org -p 2220
   ```

6. **Retrieve the password:** Once the shell prompt changed to `bandit14@bandit`: I accessed the password file for the next level.
   
```bash
   cat /etc/bandit_pass/bandit14
   # Output: WBBN9b6cn96Dx7pt9S0pS09vS7UURSYW 
   ```

---

### Lessons Learned

#### SSH Identity Files (-i)
The `-i` flag allows you to authenticate using a Public Key instead of a password. In this scenario: the private key acts as a digital signature that matches a public key already authorized on the server for `bandit14`.

#### Private Key Permissions
Security is a core requirement for SSH. If a private key file has permissions that allow group or world reading (like `644`): the SSH client will throw a `WARNING: UNPROTECTED PRIVATE KEY FILE!` and refuse to connect. The standard practice is to use `chmod 600`: which grants read/write access only to the owner.



#### Formatting and Libcrypto Errors
Private keys are extremely sensitive to formatting. If the header `-----BEGIN RSA PRIVATE KEY-----` or the footer are missing dashes: or if there are hidden spaces/line breaks: you will encounter a `libcrypto` error. The file must be a clean: exact copy of the generated key.

#### Writable Directories in Linux
On many restricted systems (like Bandit): you cannot create files in your home directory. The `/tmp` directory is a global "scratchpad" where any user can create temporary files and folders: making it essential for tasks involving intermediate data processing.

> **Pro Tip:** When you see a "Permission denied" error while trying to save a file: always try moving to `/tmp` and creating a subdirectory there. Just remember that `/tmp` is shared: so use unique names for your folders to avoid collisions with other users.
```
