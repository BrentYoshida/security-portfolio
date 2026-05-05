# Bandit Level 14

### Goal
The goal of this level is to submit the password of the current level (`bandit14`) to a service running on `localhost` at port `30000`. If the correct password is provided: the service will return the password for `bandit15`.

### Commands Used
* `cat`
* `nc` (netcat)
* `echo`

### Walkthrough
1. **Retrieve current password:** I started by reading the password for the current level from the standard location.
   ```bash
   cat /etc/bandit_pass/bandit14
   # Output: MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS
   ```

2. **Identify the obstacle:** I initially tried to connect via SSH to port 30000: but the connection was immediately closed because port 30000 is a raw network socket: not an SSH server.

3. **Establish a network connection:** I used `nc` (netcat) to connect to the service running locally.
   ```bash
   nc localhost 30000
   ```

4. **Submit the password:** Once the connection was open: I pasted the password for `bandit14` and pressed Enter.
   ```text
   MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS
   Correct!
   8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo
   ```

5. **Alternative "One-Liner" Method:** I also explored using a pipe to send the password directly to the service in a single command.
   ```bash
   echo "MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS" | nc localhost 30000
   ```

---

### Lessons Learned

#### Netcat (nc)
Netcat is often called the "Swiss Army Knife" of networking. It allows you to read and write data across network connections using TCP or UDP.
* **Usage:** `nc [hostname] [port]`
* **Purpose:** In this level: it allowed me to interact with a non-SSH service that simply waits for a specific string of text to trigger a response.

#### Redirection and Pipes
Understanding the flow of data is critical in Linux.
* **The Pipe (`|`):** Connects the standard output of one command to the standard input of another. 
* **Data Flow:** Data moves from **Left to Right**. To send a password to `nc`: the command generating the password (`echo`) must be on the left side of the pipe.
* **Common Mistake:** Running `nc localhost 30000 | echo "password"` fails because it sends the server's output to the `echo` command: rather than sending the password to the server.

#### Port Protocols
Not every open port on a server is an SSH port.
* **Port 22:** Standard for SSH.
* **Port 30000:** In this case: a raw TCP socket.
* Trying to use the wrong protocol (SSH) on a raw port will result in a "Connection closed" or "Protocol mismatch" error.

> **Pro Tip:** When you encounter an unknown port: you can use `nc -v localhost <port>` to get a "verbose" connection. If it connects: try typing something to see how the service responds. Many simple capture-the-flag (CTF) challenges use these raw sockets to test your ability to handle basic network I/O.
```
