# 🔐 BCS303-linux-security-project

> Hands-on cybersecurity project covering cryptography, buffer overflow defense, MFA, system hardening, and firewall configuration — all executed on Kali Linux.

---

## 📌 Overview

This project was built for **BCS 303 – Security Principles and Practices** at Canadian University Dubai. It is organized into three modules, each targeting a distinct domain of practical cybersecurity:

- **Module 1** — Cryptographic techniques (symmetric, asymmetric, TLS/SSL)
- **Module 2** — Defensive security (buffer overflow mitigation, SSH MFA)
- **Module 3** — OS hardening and advanced firewall configuration with `iptables`

All tasks were performed on **Kali Linux** using pre-installed security tools.

---

## 🗂️ Project Structure

```
BCS303-linux-security-project/
│
├── module1/
│   ├── encrypted_file.enc            # AES-256 encrypted file
│   ├── encrypted_file_des.enc        # DES encrypted file
│   ├── encrypted_file_3des.enc       # 3DES encrypted file
│   ├── private_key.pem               # RSA private key
│   ├── public_key.pem                # RSA public key
│   └── report_module1.md             # Cryptography analysis & findings
│
├── module2/
│   ├── vuln.c                        # Original vulnerable C program
│   ├── vuln_fixed.c                  # Patched C program with bounds checking
│   └── report_module2.md             # Buffer overflow & MFA analysis
│
├── module3/
│   ├── iptables_rules.sh             # All configured firewall rules
│   └── report_module3.md             # Hardening steps & firewall documentation
│
└── README.md
```

---

## 🔑 Module 1 — Application of Cryptographic Techniques
**Owner: Hassan Mujtaba (20220002085)**

### Task 1: Symmetric Encryption & Decryption
Encrypted a file using three algorithms via OpenSSL and compared their performance:

```bash
# Encrypt
openssl enc -aes-256-cbc -in thumb-1920-1219983.png -out encrypted_file.enc -k password
openssl enc -des-cbc -in thumb-1920-1219983.png -out encrypted_file_des.enc -k password
openssl enc -des-ede3-cbc -in thumb-1920-1219983.png -out encrypted_file_3des.enc -k password

# Decrypt
openssl enc -aes-256-cbc -d -in encrypted_file.enc -out decrypted_file_aes -k password
openssl enc -d -des-cbc -in encrypted_file_des.enc -out decrypted_file_des -k password
openssl enc -d -des-ede3-cbc -in encrypted_file_3des.enc -out decrypted_file_3des -k password
```

**Algorithm Comparison:**

| Algorithm | Speed | Security |
|---|---|---|
| DES | Fastest | Weakest — deprecated |
| AES-256 | Moderate | Strongest — industry standard |
| 3DES | Slowest | Moderate — more secure than DES but outclassed by AES |

### Task 2: Asymmetric Encryption & Key Exchange
Generated RSA-2048 key pairs and encrypted/decrypted messages using public/private keys:

```bash
# Generate keys
openssl genrsa -out private_key.pem 2048
openssl rsa -in private_key.pem -pubout -out public_key.pem

# Encrypt with public key / Decrypt with private key
openssl rsautl -encrypt -inkey public_key.pem -pubin -in message.txt -out encrypted_msg
openssl rsautl -decrypt -inkey private_key.pem -in encrypted_msg -out decrypted_msg
```

### Task 3: TLS/SSL Implementation
Generated a self-signed SSL certificate and configured Nginx to serve over HTTPS:

```bash
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
```

Nginx was configured to listen on port 443 with the generated certificate, securing data in transit against eavesdropping via TLS handshake-based key exchange.

---

## 🛡️ Module 2 — Defensive Security against Program & Web Threats
**Owner: Leanne Jessica Rodrigo (20210001983)**

### Task 1: Buffer Overflow Analysis & Defense

**Vulnerable C program (intentional):**
```c
void vulnerable_function(char *input) {
    char buffer[10];
    strcpy(buffer, input);  // No bounds check — vulnerable
}
```

Compiled without stack protection and analyzed in GDB:
```bash
gcc -fno-stack-protector -o vuln vuln.c
gdb ./vuln
```
A string longer than 10 characters caused a segmentation fault, with corrupted values observed in the instruction pointer and base pointer registers.

**Mitigation applied:**
```c
strncpy(buffer, input, sizeof(buffer) - 1);
buffer[sizeof(buffer) - 1] = '\0';  // Bounds-safe copy
```

Recompiled and verified — overflow no longer possible.

### Task 2: Multi-Factor Authentication (MFA) via SSH

Configured Google Authenticator as a second factor for SSH login:

```bash
# Install PAM module
sudo apt install libpam-google-authenticator
google-authenticator

# Enable in SSH config
sudo nano /etc/ssh/sshd_config
# Set: ChallengeResponseAuthentication yes

# Add to PAM
sudo nano /etc/pam.d/sshd
# Add: auth required pam_google_authenticator.so

# Restart SSH
sudo systemctl restart sshd
```

Login now requires both the SSH password and a time-based OTP (TOTP) from the Google Authenticator app — refreshed every 30 seconds. Emergency scratch codes were generated as backup.

---

## 🧱 Module 3 — OS Security and Firewall Configuration
**Owner: Yasmin Akram Issa (20230003798)**

### Task 1: Linux System Hardening with Lynis

```bash
sudo apt install lynis
sudo lynis audit system
```

Implemented recommendations from the audit:
```bash
sudo systemctl disable ftp          # Disable unused FTP service
sudo chmod 600 /etc/shadow          # Restrict password file access
```

ClamAV antivirus was also installed and configured for ongoing protection:
```bash
sudo apt install clamav clamav-daemon
sudo systemctl enable clamav-freshclam
sudo systemctl start clamav-freshclam
```

### Task 2: Advanced iptables Firewall + Dynamic Blocklist

Set default policy to DROP all traffic:
```bash
sudo iptables -F
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT DROP
```

Specific rules applied:
```bash
# Allow SSH from trusted IP only
sudo iptables -A INPUT -p tcp --dport 22 -s 192.168.1.10 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Block specific attacker IP
sudo iptables -A INPUT -s 192.168.1.50 -j DROP

# Log dropped packets
sudo iptables -A INPUT -j LOG --log-prefix "IPTABLES-DROP: "
```

Firewall rules were tested using `netcat`:
```bash
nc -zv <server_ip> 22          # Test allowed port
nc -zv <server_ip> 22 -s 192.168.1.50   # Test blocked IP
```

---

## ⚙️ Tools Used

| Tool | Purpose |
|---|---|
| `openssl` | Symmetric & asymmetric encryption, TLS certificate generation |
| `gpg` | Key generation and asymmetric encryption |
| `nginx` | TLS/SSL web server configuration |
| `gcc` / `gdb` | Buffer overflow compilation and analysis |
| `libpam-google-authenticator` | SSH MFA setup |
| `lynis` | Linux system security audit |
| `iptables` | Firewall rule configuration |
| `clamav` | Antivirus scanning and real-time protection |
| `netcat` | Firewall rule testing and port validation |

---

## 👥 Team

| Student | ID | Module |
|---|---|---|
| **Hassan Mujtaba** | 20220002085 | Module 1 — Cryptographic Techniques |
| **Leanne Jessica Rodrigo** | 20210001983 | Module 2 — Defensive Security & MFA |
| **Yasmin Akram Issa** | 20230003798 | Module 3 — OS Hardening & Firewall |

---

## 🏫 Course Information

**BCS 303 – Security Principles and Practices**  
School of Engineering, Applied Sciences, and Technology  
Canadian University Dubai — Summer I, 2025

---

## ⚠️ Notes!

- All tasks were performed inside a **Kali Linux virtual machine** — do not run these commands on a production system without understanding their impact
- The `iptables` rules in this project drop all traffic by default; adapt rules to your specific environment before applying
- The vulnerable C program (`vuln.c`) is included for **educational purposes only**
- Private keys and `.env` files are **gitignored** and must never be committed
