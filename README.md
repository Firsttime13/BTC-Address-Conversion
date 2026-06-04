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
