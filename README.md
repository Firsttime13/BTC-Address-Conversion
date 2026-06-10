# Bitcoin Puzzle Tools

Pure-Python scripts for handling legacy Bitcoin addresses and keys (educational use only).

- `btc_address_checksum.py`: Validate P2PKH address checksum.
- `wif_to_hex.py`: Decode WIF → hex private key.
- `hex_to_wif.py`: Encode hex private key → WIF.

**Never use with real private keys online. For puzzle addresses like those in bitcoin-puzzle-unsolved-20260123.csv, checksums are valid but private keys remain unknown for 71+ bits.**

Commit.

Running on Ubuntu 24.04+
Bash
      sudo apt update && sudo apt install python3 git -y  \\
      git clone https://github.com/(Repo-name).git  # or your repo name/path  \\
      cd Repo-name                                  # or your repo name/path  \\
      
python3 btc_address_checksum.py   # or whichever script

(Use python3 -m venv venv && source venv/bin/activate for isolation.)


# BTC Address Toolkit
### Three-Script Bitcoin Key Utilities for use with keyhunt-cyclone

A pipeline of three pure-Python scripts that decode Bitcoin addresses, extract
cryptographic components, and convert private keys to WIF format.  Designed as
a companion to [keyhunt-cyclone](https://github.com/Firsttime13/).

---

## Table of Contents
1. [Scripts overview](#scripts-overview)
2. [How the pipeline works](#how-the-pipeline-works)
3. [Installation](#installation)
   - [Ubuntu 24.04+](#ubuntu-2404)
   - [Windows 10](#windows-10)
4. [Running the scripts](#running-the-scripts)
5. [Test: address 1JCeMgVeDzLdxz3G5vRin2ydNxUp6E5yFf](#test-run)
6. [Adding to your GitHub repository](#adding-to-your-github-repository)
7. [Integration with keyhunt-cyclone](#integration-with-keyhunt-cyclone)
8. [Security notes](#security-notes)

---

## Scripts Overview

| File | Purpose |
|---|---|
| `btc_checksum.py` | Decodes a Base58Check address and verifies/extracts the 4-byte checksum + 20-byte hash160 |
| `btc_key_lookup.py` | Uses the decoded data; queries the blockchain API to retrieve the public key (only available for spent addresses) |
| `btc_wif_converter.py` | Converts a raw 256-bit hex private key → WIF (compressed & uncompressed), derives public keys and P2PKH addresses |
| `btc_pipeline_test.py` | End-to-end integration test using the test address and a BIP-standard key vector |

---

## How the Pipeline Works

```
Bitcoin Address (Base58Check)
        │
        ▼
 btc_checksum.py
  • Base58 decode (25 bytes)
  • Extract version byte, hash160, embedded checksum
  • Recompute checksum via double-SHA256
  • Confirm validity
        │
        ▼  hash160 (20 bytes)
 btc_key_lookup.py
  • Query blockchain.info API
  • If address has been spent → public key is revealed
  • Verify pubkey → hash160 match
        │
        ▼  (if private key found via keyhunt-cyclone)
 btc_wif_converter.py
  • secp256k1 scalar multiplication (pure Python)
  • Derive compressed / uncompressed public keys
  • Encode to WIF Uncompressed (5...)
  • Encode to WIF Compressed   (K/L...)
  • Derive P2PKH Bitcoin addresses
```

**Why can't the address be reversed to get the private key?**
A Bitcoin address is `Base58Check(0x00 + RIPEMD160(SHA256(pubkey)))`.
Both SHA256 and RIPEMD160 are one-way functions — reversing them is
computationally infeasible. The private key must be *found* by searching the
key space (e.g., with keyhunt-cyclone); it cannot be derived from the address.

---

## Installation

### Ubuntu 24.04+

**Step 1 – Update packages and install Python**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip git
```

**Step 2 – Verify Python version (must be 3.8+)**
```bash
python3 --version
# Expected: Python 3.12.x or higher
```

**Step 3 – Clone your GitHub repository**
```bash
git clone https://github.com/Firsttime13/<YOUR_REPO_NAME>.git
cd <YOUR_REPO_NAME>
```

**Step 4 – (Optional) Install the `requests` library**
The scripts use Python's built-in `urllib` for blockchain queries.
No external packages are required.  If you prefer `requests`:
```bash
pip3 install requests --break-system-packages
```

**Step 5 – Make scripts executable (optional)**
```bash
chmod +x btc_checksum.py btc_key_lookup.py btc_wif_converter.py
```

---

### Windows 10

**Step 1 – Install Python 3.10+**
- Go to https://www.python.org/downloads/
- Download the latest Python 3.x installer
- ✅ Check **"Add Python to PATH"** before clicking Install
- Click **Install Now**

**Step 2 – Verify installation (open Command Prompt or PowerShell)**
```cmd
python --version
pip --version
```

**Step 3 – Install Git for Windows**
- Go to https://git-scm.com/download/win
- Download and run the installer (defaults are fine)

**Step 4 – Clone your repository**
```cmd
git clone https://github.com/Firsttime13/<YOUR_REPO_NAME>.git
cd <YOUR_REPO_NAME>
```

**Step 5 – (Optional) Install requests**
```cmd
pip install requests
```

---

## Running the Scripts

All three scripts accept an argument or will prompt if no argument is given.

### Script 1 – Checksum Verification

```bash
# Ubuntu
python3 btc_checksum.py 1JCeMgVeDzLdxz3G5vRin2ydNxUp6E5yFf

# Windows
python btc_checksum.py 1JCeMgVeDzLdxz3G5vRin2ydNxUp6E5yFf
```

### Script 2 – Key Lookup

```bash
# Ubuntu
python3 btc_key_lookup.py 1JCeMgVeDzLdxz3G5vRin2ydNxUp6E5yFf

# Windows
python btc_key_lookup.py 1JCeMgVeDzLdxz3G5vRin2ydNxUp6E5yFf
```

### Script 3 – WIF Converter (requires a known private key hex)

```bash
# Ubuntu – replace <HEX> with a 64-character hex private key
python3 btc_wif_converter.py <HEX_PRIVATE_KEY>

# Windows
python btc_wif_converter.py <HEX_PRIVATE_KEY>
```

### Run the full pipeline test

```bash
# Ubuntu
python3 btc_pipeline_test.py

# Windows
python btc_pipeline_test.py
```

---

## Test Run

Test address: **`1JCeMgVeDzLdxz3G5vRin2ydNxUp6E5yFf`**

### Expected output – Script 1 (`btc_checksum.py`)

```
============================================================
  BITCOIN ADDRESS CHECKSUM REPORT
============================================================
  Address        : 1JCeMgVeDzLdxz3G5vRin2ydNxUp6E5yFf
  Raw (hex)      : 00bcade5c8ddf6a571e438575771ce766ae7a0194833227c02
  Version byte   : 0x00
  Address type   : P2PKH (Mainnet – starts with '1')
  Payload (hex)  : 00bcade5c8ddf6a571e438575771ce766ae7a01948
  PubKey hash    : bcade5c8ddf6a571e438575771ce766ae7a01948
------------------------------------------------------------
  Embedded CSum  : 33227c02
  Computed CSum  : 33227c02
------------------------------------------------------------
  Checksum       : ✅  VALID
============================================================
```

**Key output values:**

| Field | Value |
|---|---|
| hash160 (RIPEMD160) | `bcade5c8ddf6a571e438575771ce766ae7a01948` |
| Checksum (4 bytes) | `33227c02` |
| Address type | P2PKH Mainnet |
| Version byte | `0x00` |

The `hash160` value is what keyhunt-cyclone compares against when scanning
the private key space.

---

### Script 3 WIF Example (BIP standard test vector)

```bash
python3 btc_wif_converter.py 0c28fca386c7a227600b2fe50b7cae11ec86d3bf1fbe471be89827e19d72aa1d
```

```
  WIF Uncompressed (5...)  : 5HueCGU8rMjxEXxiPuD5BDku4MkFqeZyd4dZ1jvhTVqvbTLvyTJ
  WIF Compressed   (K/L..) : KwdMAjGmerYanjeui5SHS7JkmpZvVipYvB2LJGU1ZxJwYvP98617
  Compressed   (33 B)      : 02d0de0aaeaefad02b8bdc8a01a1b8b11c696bd3d66a2c5f10780d95b7df42645c
  From compressed pubkey   : 1LoVGDgRs9hTfTNJNuXKSpywcbdvwRXpmK
  From uncompressed pubkey : 1GAehh7TsJAHuUAeKZcXf5CnwuGuGgyX2S
```

---

## Adding to Your GitHub Repository

### Step-by-step (Ubuntu or Windows Git Bash)

**Step 1 – Navigate into your local repository clone**
```bash
cd /path/to/your/cloned/repo
# or on Windows:
cd C:\Users\YourName\<YOUR_REPO_NAME>
```

**Step 2 – Create a subdirectory for the toolkit (recommended)**
```bash
mkdir btc-address-toolkit
cd btc-address-toolkit
```

**Step 3 – Copy the four script files into this folder**
```bash
# Ubuntu: copy from wherever you saved them
cp /path/to/btc_checksum.py .
cp /path/to/btc_key_lookup.py .
cp /path/to/btc_wif_converter.py .
cp /path/to/btc_pipeline_test.py .

# Windows (PowerShell):
copy C:\path\to\btc_checksum.py .
copy C:\path\to\btc_key_lookup.py .
copy C:\path\to\btc_wif_converter.py .
copy C:\path\to\btc_pipeline_test.py .
```

**Step 4 – Check git status**
```bash
cd ..   # go back to repo root
git status
```
You should see the new files listed as "untracked".

**Step 5 – Stage the new files**
```bash
git add btc-address-toolkit/
```

**Step 6 – Commit with a descriptive message**
```bash
git commit -m "Add BTC address toolkit: checksum, key lookup, WIF converter"
```

**Step 7 – Push to GitHub**
```bash
git push origin main
# If your default branch is called 'master':
# git push origin master
```

**Step 8 – Verify on GitHub**
- Open https://github.com/Firsttime13/<YOUR_REPO_NAME>
- You should see the `btc-address-toolkit/` folder with all four files

---

### If you don't have a repo yet (create one first)

```bash
# 1. On GitHub.com: click "New repository", name it, create it

# 2. On your machine:
git init
git remote add origin https://github.com/Firsttime13/<REPO_NAME>.git
git add btc_checksum.py btc_key_lookup.py btc_wif_converter.py btc_pipeline_test.py README.md
git commit -m "Initial commit: BTC address toolkit"
git branch -M main
git push -u origin main
```

---

## Integration with keyhunt-cyclone

When keyhunt-cyclone finds a candidate private key (in hex format), run
it through Script 3 to:

1. Generate both WIF formats for wallet import
2. Derive the corresponding Bitcoin addresses
3. Confirm which address format (compressed vs uncompressed) matches your target

```bash
# Example workflow after keyhunt-cyclone reports a hit:
python3 btc_wif_converter.py <FOUND_HEX_KEY>

# The output will show:
#   From compressed pubkey   : 1Xxx...   ← check if this matches your target
#   From uncompressed pubkey : 1Yyy...   ← or this one
```

The `hash160` from Script 1 (`bcade5c8ddf6a571e438575771ce766ae7a01948` for
the test address) is the exact 20-byte value keyhunt-cyclone uses internally
as a search target.

---

## Security Notes

- These scripts **cannot crack** Bitcoin addresses. They are utilities for
  key format conversion and address verification.
- The private key space is 2²⁵⁶ ≈ 10⁷⁷.  Finding a specific key requires
  purpose-built search tools operating on constrained ranges.
- **Never enter a private key that controls real funds** on an untrusted
  machine or share it publicly.
- Scripts 1 and 2 only read public blockchain data and perform local hashing.
  No private data is transmitted.

---

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| Python 3.8+ | Yes | Standard library only |
| hashlib | Yes | Built-in |
| urllib | Yes | Built-in |
| requests | No | Optional alternative for API calls |

No `pip install` is required to run these scripts.

---

*Part of the [Firsttime13](https://github.com/Firsttime13/) Bitcoin toolkit.*
