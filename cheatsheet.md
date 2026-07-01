# Phase 1 Complete Cheat Sheet 🛡️
## Computer Fundamentals — Quick Revision Guide

---

## 1.1 — What is a Computer?

| Component | Job |
|---|---|
| CPU | Executes instructions |
| RAM | Temporary fast memory |
| Storage | Permanent memory |
| Motherboard | Connects everything |
| GPU | Graphics + fast calculations |
| NIC | Network connection |
| PSU | Powers everything |

**Hacking relevance:** GPU cracks passwords. NIC has MAC address (fakeable). Storage holds sensitive files.

---

## 1.2 — CPU

| Term | Meaning |
|---|---|
| GHz | CPU speed (billions of cycles/second) |
| Core | Independent processing unit |
| Register | Tiny ultra-fast storage inside CPU |
| EIP/RIP | Holds address of next instruction |
| User Mode | Limited access for normal programs |
| Kernel Mode | Full access for OS |

**Key attacks:**
- Buffer overflow → overwrites **EIP** → redirects execution
- Privilege escalation → moves from **User Mode** to **Kernel Mode**

**Defenses:** ASLR, DEP/NX

---

## 1.3 — RAM

| Term | Meaning |
|---|---|
| RAM | Fast temporary memory, wiped on shutdown |
| Stack | Memory for function calls |
| Heap | Dynamic memory for running programs |
| Virtual Memory | Disk space acting as backup RAM |
| LSASS | Windows process holding login credentials |
| Pagefile/Swap | Windows/Linux name for virtual memory file |

**Key attacks:**
- **Mimikatz** → dumps passwords from LSASS
- **Cold boot attack** → extracts keys from RAM after shutdown
- **Buffer overflow** → overflows stack

**Defenses:** Credential Guard, ASLR, DEP

**Tool:** Volatility (memory forensics)

---

## 1.4 — Storage

| Type | Speed | Notes |
|---|---|---|
| HDD | ~150 MB/s | Magnetic, moving parts, cheap |
| SSD (SATA) | ~500 MB/s | Flash, silent, fast |
| SSD (NVMe) | ~3500 MB/s | Ultra-fast, direct to motherboard |

**File Systems:**

| File System | OS |
|---|---|
| NTFS | Windows |
| ext4 | Linux |
| FAT32 | USB drives |

**Critical fact:** Deleting a file only removes its directory entry. **Data remains until overwritten.**

**Key attacks:**
- Deleted file recovery → **Autopsy, PhotoRec**
- SAM database theft → Windows password hashes
- Log deletion → hide attack evidence

**Secure deletion:** `shred -u -z -n 3 filename`

---

## 1.5 — BIOS/UEFI

| Term | Meaning |
|---|---|
| BIOS | Old firmware, starts computer |
| UEFI | Modern BIOS, faster + more secure |
| POST | Hardware self-check at startup |
| Secure Boot | Verifies bootloader signature |
| Bootloader | Loads OS from storage |
| CMOS | Chip storing BIOS settings |
| VT-x/AMD-V | Virtualization setting needed for VirtualBox |

**Boot order:** BIOS → POST → Bootloader → OS

**Key attacks:**
- **Bootkit** → infects boot process before OS loads
- **UEFI Rootkit** → survives OS reinstall + drive replacement
- **CMOS reset** → removes BIOS password physically

**Important for lab:** Enable **VT-x/AMD-V** in UEFI for VirtualBox

---

## 1.6 — Operating Systems

| OS | Used By | Kernel |
|---|---|---|
| Windows | Home/Enterprise | NT Kernel |
| Linux | Servers/Hackers | Linux Kernel |
| macOS | Apple users | XNU Kernel |

**Linux distros for hacking:**

| Distro | Purpose |
|---|---|
| Kali Linux | Penetration testing (your main OS) |
| Parrot OS | Lightweight alternative |
| BlackArch | Advanced, massive tool library |

**Key concept:** Kernel Mode = full control. Getting code into kernel = total compromise.

**First commands after access:**
```bash
whoami    # Who am I?
id        # What groups/privileges do I have?
```

**Famous vulnerabilities:**
- EternalBlue → WannaCry ransomware (Windows)
- Dirty COW → Linux privilege escalation
- Shellshock → Remote code execution via Bash

---

## 1.7 — File Systems

**Critical Linux paths:**

| Path | Contents |
|---|---|
| /etc/passwd | User account info (readable by all) |
| /etc/shadow | Password hashes (root only) ⭐ |
| /var/log/ | System logs — attack evidence |
| /tmp/ | Temp files — malware drops here |
| /home/ | User home directories |

**Critical Windows paths:**

| Path | Contents |
|---|---|
| C:\Windows\System32 | Core Windows files |
| C:\Users\[user]\AppData | Hidden — malware configs |
| C:\Temp | Malware drop location |

**Key concepts:**

| Term | Meaning |
|---|---|
| ADS | Hidden data stream in NTFS files — used to hide malware |
| Timestomping | Attacker modifies file timestamps to hide activity |
| Inode | Linux structure storing file metadata |
| Journaling | Records changes before making them |

**Key attacks:** ADS abuse, timestomping, /etc/shadow theft, log deletion

---

## 1.8 — Processes

| Term | Meaning |
|---|---|
| Process | Running program in RAM |
| Program | Inactive file on storage |
| PID | Unique number identifying process |
| Thread | Lightweight task inside a process |
| Parent process | Process that created another |
| Child process | Process created by parent |

**Important Windows processes:**

| Process | Purpose | Hacking Relevance |
|---|---|---|
| lsass.exe | Stores credentials | Mimikatz target ⭐ |
| svchost.exe | Hosts services | Malware disguises as this |
| explorer.exe | Windows GUI | Common injection target |
| powershell.exe | PowerShell | Heavily abused by malware |

**Red flag:** svchost.exe running from C:\Temp → it's malware. Legit svchost only runs from System32.

**Key attacks:**
- **Process injection** → inject code into legitimate process
- **Process hollowing** → replace legitimate process with malware
- **LSASS dumping** → steal credentials from memory
- **Token impersonation** → steal process privileges

**Commands:**
```bash
# Linux
ps aux          # List processes
pstree          # Parent-child tree
kill [PID]      # Kill process

# Windows
tasklist        # List processes
taskkill /PID [n] /F   # Kill process
```

---

## 1.9 — Memory Management

**Memory layout (per process):**
```
Stack  → Function calls, local variables (fixed size)
Heap   → Dynamic memory (grows/shrinks)
Data   → Global variables
Code   → Program instructions
```

**Buffer overflow:**
```
Too much data → overflows stack → overwrites EIP → attacker redirects execution
```

**Key concepts:**

| Term | Meaning |
|---|---|
| Buffer overflow | Writing past memory boundary |
| Heap spray | Filling heap with malicious code |
| Use-after-free | Using freed memory — exploitable |
| Fileless malware | Runs only in RAM, never touches disk |
| Page fault | Needed memory page not in RAM |
| MMU | CPU component translating virtual to physical addresses |

**Defenses:**

| Defense | What It Does |
|---|---|
| ASLR | Randomizes memory addresses |
| DEP/NX | Prevents data pages from executing |
| Stack Canary | Detects stack overflow attempts |

**Tool:** **Volatility** → memory forensics → catches fileless malware

---

## 1.10 — Users and Permissions

**Linux user types:**

| Type | Privilege | Symbol |
|---|---|---|
| Root | Complete control | # |
| Regular user | Limited | $ |
| Service account | Runs services | — |

**Root UID is always 0.**

**Linux permissions:**
```
-rwxr-xr--
 │││├──┤├──┤
 │││ Group  Others
 ││└── Owner
 │└── File type
```

**Numeric permissions:**

| Number | Permission |
|---|---|
| 7 | rwx (read+write+execute) |
| 6 | rw- (read+write) |
| 5 | r-x (read+execute) |
| 4 | r-- (read only) |
| 0 | --- (no access) |

**Common permission sets:**

| chmod | Meaning | Use |
|---|---|---|
| 777 | Everyone full access | ❌ Dangerous |
| 755 | Owner full, others read+execute | Normal executables |
| 644 | Owner rw, others read | Normal files |
| 600 | Owner rw only | SSH keys, private files |

**Special permissions:**

| Permission | Meaning | Hacking Use |
|---|---|---|
| SUID | File runs as owner's privilege | Privesc if owned by root ⭐ |
| SGID | File runs as group's privilege | Similar to SUID |
| Sticky bit | Only owner can delete their files | /tmp protection |

**Key commands:**
```bash
whoami          # Current user
id              # User ID + groups
sudo -l         # What can I run as root? ⭐ (run this first!)
chmod 755 file  # Change permissions
chown user file # Change owner

# Find SUID files (privilege escalation hunting)
find / -perm -4000 2>/dev/null

# Find world-writable files (misconfiguration hunting)
find / -perm -o+w -type f 2>/dev/null
```

**Windows permissions:**

| Permission | Meaning |
|---|---|
| Full Control | Everything |
| Modify | Read, write, delete |
| Read & Execute | View and run |
| Read | View only |
| Write | Create and modify |

**Key files:**

| File | Contents |
|---|---|
| /etc/passwd | Usernames, UIDs (readable by all) |
| /etc/shadow | Password hashes (root only) ⭐ |

**Resource:** GTFOBins → lists exploitable Linux binaries with SUID

---

## Master Commands Reference

**Linux:**
```bash
# System info
uname -a              # OS info
lscpu                 # CPU info
free -h               # RAM info
df -h                 # Disk info
lsblk                 # Storage devices

# Users
whoami                # Current user
id                    # User + groups
sudo -l               # Sudo permissions
cat /etc/passwd       # All users
sudo cat /etc/shadow  # Password hashes

# Files
ls -la                # List with hidden files
find / -perm -4000    # Find SUID files
chmod 755 file        # Change permissions
chown user file       # Change owner

# Processes
ps aux                # All processes
pstree                # Process tree
kill [PID]            # Kill process
top / htop            # Real-time processes

# Memory
free -h               # RAM usage
cat /proc/meminfo     # Detailed memory
```

**Windows (PowerShell):**
```powershell
whoami              # Current user
whoami /priv        # Privileges
whoami /groups      # Group memberships
net user            # All users
net localgroup Administrators  # Admin group members
tasklist            # Running processes
systeminfo          # Full system info
```

---

## Top 10 Things to Remember from Phase 1

1. **Deleting a file doesn't erase it** — data stays until overwritten
2. **RAM contains passwords, keys, and tokens** — attackers dump it
3. **SUID files owned by root** = potential privilege escalation
4. **sudo -l** = first command after getting Linux shell access
5. **/etc/shadow** = Linux password hashes = prime target
6. **svchost.exe from C:\Temp** = malware, not legitimate Windows process
7. **UEFI rootkits survive** OS reinstall and drive replacement
8. **Buffer overflow overwrites EIP** = attacker controls execution
9. **Fileless malware** lives in RAM only — needs memory forensics to detect
10. **chmod 777** = dangerous — never use on sensitive files

---

## Key Tools Introduced in Phase 1

| Tool | Purpose |
|---|---|
| **Volatility** | Memory forensics — finds fileless malware |
| **Mimikatz** | Extracts passwords from RAM (authorized labs) |
| **Autopsy/PhotoRec** | Recovers deleted files |
| **GTFOBins** | Lists exploitable Linux binaries |
| **shred** | Securely deletes files |

---
