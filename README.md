# Secure Encrypt - Offline SMS Encryption Tool

A simple, secure desktop app for encrypting and decrypting short messages using **Argon2id + AES-256-GCM** — military-grade encryption.

Perfect for secure communication via SMS when the internet is down, blocked, or monitored.

### Features
- Single executable file (no installation needed)
- Works completely offline
- Nothing saved to disk
- Extremely resistant to brute-force attacks (Argon2id with up to 1 GiB memory)
- Beautiful dark-themed GUI
- Supports **Linux (Ubuntu & others)** and **Windows 10/11**

## Quick Start (From Source)

### 1. Install Go
- **Linux/Ubuntu**:
  ```bash
  sudo apt update && sudo apt install golang-go -y

  Windows:
Download and run the installer from: https://go.dev/dl/
(Make sure to check "Add to PATH" during installation)

Build & Run:

git clone https://github.com/erfankazemi-byte/secure-encrypt-go.git
cd secure-encrypt-go
go build -o secure-encrypt

LINUX:
./secure-encrypt

Windows:
Double-click secure-encrypt.exe or run in Command Prompt/PowerShell: 

secure-encrypt.exe

installng the app system wide(optional):
sudo cp secure-encrypt /usr/local/bin/
sudo tee /usr/share/applications/secure-encrypt.desktop > /dev/null <<EOF
[Desktop Entry]
Name=Secure Encrypt
Comment=Offline secure messaging tool (Argon2id + AES-256-GCM)
Exec=/usr/local/bin/secure-encrypt
Icon=locked
Terminal=false
Type=Application
Categories=Utility;Security;
Keywords=encrypt;decrypt;sms;privacy;offline;
EOF

Now search for "Secure Encrypt" in your applications menu.


Windows

Copy secure-encrypt.exe to any folder (e.g., C:\Tools\)
Optional: Create a desktop shortcut
Optional: Right-click shortcut → Properties → Change Icon → pick a lock/shield icon

Why This Tool?
Built for journalists, activists, and anyone needing private communication in restrictive environments.
No telemetry, no logs, no internet access — pure local encryption.
License
MIT License — free to use, modify, share, and distribute.
Made with care for privacy and human rights.
