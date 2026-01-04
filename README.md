**Offline SMS Encryption Tool**

A lightweight desktop application for encrypting and decrypting short messages using **Argon2id + AES-256-GCM**.

Designed for secure communication when the internet is unavailable, restricted, or untrusted.

---

## Highlights

* Works **fully offline**
* **Single executable** — no installation required
* **Nothing written to disk**
* Strong protection against brute-force attacks
  *(Argon2id with up to 1 GiB memory)*
* Clean, dark-themed GUI
* Linux (Ubuntu & others) and Windows 10/11 support

---

## Use Cases

* Secure SMS communication
* Privacy-focused messaging
* Environments with censorship or surveillance
* Situations where internet access is unreliable or blocked

---

## Quick Start (Build from Source)

### 1. Install Go

**Linux (Ubuntu / Debian):**

```bash
sudo apt update && sudo apt install golang-go -y
```

**Windows:**
Download from [https://go.dev/dl/](https://go.dev/dl/)
(Check **“Add to PATH”** during installation)

---

### 2. Build the App

```bash
git clone https://github.com/erfankazemi-byte/secure-encrypt-go.git
cd secure-encrypt-go
go build -o secure-encrypt
```

---

### 3. Run

**Linux:**

```bash
./secure-encrypt
```

**Windows:**

```powershell
secure-encrypt.exe
```

(or double-click the file)

---

## Optional: System-Wide Install (Linux)

```bash
sudo cp secure-encrypt /usr/local/bin/
sudo tee /usr/share/applications/secure-encrypt.desktop > /dev/null <<EOF
[Desktop Entry]
Name=Secure Encrypt
Comment=Offline secure messaging tool
Exec=/usr/local/bin/secure-encrypt
Icon=locked
Terminal=false
Type=Application
Categories=Utility;Security;
Keywords=encrypt;decrypt;sms;privacy;offline;
EOF
```

You can now launch **Secure Encrypt** from the applications menu.

---

## Optional: Windows Convenience

* Move `secure-encrypt.exe` anywhere (e.g. `C:\Tools\`)
* Create a desktop shortcut
* (Optional) Change the shortcut icon to a lock or shield

---

## Security Philosophy

* No telemetry
* No logging
* No network access
* All encryption happens locally

Built with a focus on privacy, resilience, and simplicity.

---

## License

MIT License — free to use, modify, and distribute.

