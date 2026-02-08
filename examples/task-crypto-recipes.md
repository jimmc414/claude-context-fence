---
description: "Cryptography recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-crypto with target, algorithm, and context included"
when-to-use: "Receives prepared arguments from task-crypto with target, algorithm, and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Cryptography & Encoding Recipes

**Tools:** md5sum, sha1sum, sha256sum, sha512sum, b2sum, cksum, rhash, base64, base32, basenc, xxd, od, hexdump, openssl, gpg, age, sops, pass, ssh-keygen, ssh-agent, ssh-copy-id, mkcert, certbot, htpasswd

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| SHA256 hash | `sha256sum file` |
| Verify checksum | `sha256sum -c checksums.txt` |
| Base64 encode | `base64 file` |
| Base64 decode | `base64 -d encoded.txt` |
| Generate SSH key | `ssh-keygen -t ed25519 -C "comment"` |
| Encrypt (symmetric) | `gpg -c file` |
| Encrypt (recipient) | `gpg -e -r recipient file` |
| Decrypt | `gpg -d file.gpg` |
| Self-signed cert | `openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes` |
| Inspect cert | `openssl x509 -in cert.pem -text -noout` |
| TLS check | `openssl s_client -connect host:443` |
| Random bytes | `openssl rand -hex 32` |
| JWT decode | `echo "$JWT" \| cut -d. -f2 \| base64 -d 2>/dev/null \| jq .` |
| HMAC-SHA256 | `echo -n "data" \| openssl dgst -sha256 -hmac "secret"` |
| Generate password | `openssl rand -base64 24` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| File checksum (standard) | sha256sum | Widely supported, good security |
| Fast file checksum | b2sum | BLAKE2, faster than SHA-256 |
| Legacy checksum | md5sum | Only for compatibility, not security |
| Simple file encryption | age | Modern, simple API |
| Full crypto suite | gpg | Keys, signing, encryption, web of trust |
| TLS certificates | openssl | Certificate generation and inspection |
| Local dev certs | mkcert | Zero-config localhost HTTPS |
| SSH key generation | ssh-keygen | Ed25519 recommended |
| Random data generation | openssl rand | Cryptographic random bytes |
| Secret management | pass / sops | Password store / encrypted config |

---

## Checksums and Hashing

### Basic Hashing
```bash
sha256sum file.txt                    # SHA-256 (recommended default)
sha512sum file.txt                    # SHA-512
sha1sum file.txt                      # SHA-1 (deprecated for security)
md5sum file.txt                       # MD5 (legacy, not collision-resistant)
b2sum file.txt                        # BLAKE2b (fast, secure)
b2sum -l 256 file.txt                 # BLAKE2b with 256-bit output
```

### Hash a String
```bash
echo -n "text" | sha256sum            # Hash string (no trailing newline)
printf '%s' "text" | sha256sum        # Alternative (safer)
echo -n "text" | md5sum               # MD5 of string
```

### Checksum Files (create and verify)
```bash
sha256sum *.tar.gz > SHA256SUMS       # Create checksum file
sha256sum -c SHA256SUMS               # Verify all checksums
sha256sum -c --quiet SHA256SUMS       # Only report failures
sha256sum -c --strict SHA256SUMS      # Fail on improperly formatted lines

# Verify single file against known hash
echo "expected_hash  filename" | sha256sum -c -
```

### Multi-Algorithm Hashing (rhash)
```bash
rhash --sha256 file.txt               # SHA-256
rhash --all file.txt                  # All supported algorithms
rhash -a --printf="%{sha-256} %{md5} %f\n" file.txt  # Custom format
rhash --sha256 -r dir/               # Recursive directory hashing
rhash -c SHA256SUMS                   # Verify checksum file
```

### Batch Hashing
```bash
find . -type f -name "*.py" -exec sha256sum {} + > checksums.txt
sha256sum file1 file2 file3 | tee checksums.txt
```

---

## Base64 and Encoding

### Base64
```bash
base64 file.txt                       # Encode file
base64 -d encoded.txt                 # Decode file
echo -n "text" | base64               # Encode string
echo "dGV4dA==" | base64 -d           # Decode string
base64 -w 0 file.txt                  # No line wrapping (single line)
base64 -w 76 file.txt                 # Wrap at 76 chars (MIME standard)
```

### Base32
```bash
base32 file.txt                       # Encode
base32 -d encoded.txt                 # Decode
echo -n "text" | base32               # Encode string
```

### Hex Encoding (xxd)
```bash
xxd file.bin                          # Hex dump with ASCII
xxd -p file.bin                       # Plain hex (no formatting)
xxd -r -p hex.txt > file.bin          # Reverse: hex to binary
xxd -l 64 file.bin                    # First 64 bytes only
xxd -s 0x100 file.bin                 # Start at offset 0x100
xxd -i file.bin                       # C include format
echo -n "text" | xxd -p               # String to hex
echo "74657874" | xxd -r -p           # Hex to string
```

### Other Encodings
```bash
od -A x -t x1z file.bin              # Hex dump with od
od -c file.bin                        # Character representation
basenc --base64url < file.txt         # URL-safe base64
basenc --base16 < file.txt            # Hex encoding
basenc --z85 < file.txt               # Z85 encoding
```

---

## SSH Key Management

### Key Generation
```bash
ssh-keygen -t ed25519 -C "user@host"              # Ed25519 (recommended)
ssh-keygen -t rsa -b 4096 -C "user@host"          # RSA 4096-bit
ssh-keygen -t ecdsa -b 521 -C "user@host"         # ECDSA 521-bit
ssh-keygen -t ed25519 -f ~/.ssh/mykey              # Custom output file
ssh-keygen -t ed25519 -f ~/.ssh/deploy -N ""       # No passphrase
ssh-keygen -t ed25519 -f ~/.ssh/secure -N "passphrase"  # With passphrase
```

### Key Operations
```bash
ssh-keygen -p -f ~/.ssh/mykey                      # Change passphrase
ssh-keygen -y -f ~/.ssh/mykey                      # Extract public key from private
ssh-keygen -l -f ~/.ssh/mykey.pub                  # Show fingerprint
ssh-keygen -l -E md5 -f ~/.ssh/mykey.pub           # MD5 fingerprint
ssh-keygen -R hostname                             # Remove from known_hosts
ssh-keygen -F hostname                             # Find in known_hosts
```

### SSH Agent
```bash
eval $(ssh-agent)                                  # Start agent
ssh-add                                            # Add default key
ssh-add ~/.ssh/mykey                               # Add specific key
ssh-add -l                                         # List loaded keys
ssh-add -L                                         # List loaded public keys
ssh-add -D                                         # Remove all keys
ssh-add -d ~/.ssh/mykey                            # Remove specific key
ssh-add -t 3600 ~/.ssh/mykey                       # Add with 1-hour timeout
```

### Deploy Public Key
```bash
ssh-copy-id user@host                              # Copy default key
ssh-copy-id -i ~/.ssh/mykey.pub user@host          # Copy specific key
ssh-copy-id -p 2222 user@host                      # Custom port

# Manual method (if ssh-copy-id unavailable)
cat ~/.ssh/mykey.pub | ssh user@host 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

### Key Format Conversion
```bash
# PEM to OpenSSH
ssh-keygen -i -m PEM -f key.pem > key_openssh

# OpenSSH to PEM
ssh-keygen -e -m PEM -f ~/.ssh/mykey > key.pem

# OpenSSH to PKCS8
ssh-keygen -e -m PKCS8 -f ~/.ssh/mykey > key.pkcs8
```

---

## GPG / PGP

### Key Generation
```bash
gpg --gen-key                                      # Interactive key generation
gpg --full-gen-key                                 # Full options (algorithm, size, expiry)
gpg --quick-gen-key "Name <email>" ed25519         # Quick Ed25519 key
gpg --quick-gen-key "Name <email>" rsa4096         # Quick RSA-4096 key
```

### Encryption
```bash
# Symmetric (password-based)
gpg -c file.txt                                    # Encrypt (creates file.txt.gpg)
gpg -c --cipher-algo AES256 file.txt               # Specify algorithm
gpg -c -o encrypted.gpg file.txt                   # Custom output name

# Asymmetric (recipient-based)
gpg -e -r "recipient@email" file.txt               # Encrypt for recipient
gpg -e -r "alice@email" -r "bob@email" file.txt    # Multiple recipients
gpg --armor -e -r "recipient" file.txt             # ASCII-armored output (.asc)
```

### Decryption
```bash
gpg -d file.txt.gpg                                # Decrypt to stdout
gpg -d -o file.txt file.txt.gpg                    # Decrypt to file
gpg --batch --passphrase "pass" -d file.gpg        # Non-interactive (use cautiously)
```

### Signing
```bash
gpg --sign file.txt                                # Sign (creates file.txt.gpg, binary)
gpg --clearsign file.txt                           # Inline signature (readable text)
gpg --detach-sign file.txt                         # Detached signature (.sig)
gpg --armor --detach-sign file.txt                 # Detached ASCII signature (.asc)
gpg -u "key-id" --sign file.txt                    # Sign with specific key
```

### Verification
```bash
gpg --verify file.txt.sig file.txt                 # Verify detached signature
gpg --verify file.txt.asc                          # Verify inline/clearsign
```

### Key Management
```bash
gpg --list-keys                                    # List public keys
gpg --list-secret-keys                             # List private keys
gpg --list-keys --keyid-format long                # Show long key IDs
gpg --fingerprint "user@email"                     # Show fingerprint

gpg --export -a "user@email" > public.asc          # Export public key (ASCII)
gpg --export-secret-keys -a "user@email" > private.asc  # Export private key
gpg --import public.asc                            # Import key
gpg --delete-key "key-id"                          # Delete public key
gpg --delete-secret-key "key-id"                   # Delete private key

gpg --edit-key "key-id"                            # Interactive key editor
# In editor: trust, expire, passwd, adduid, save
```

### Keyservers
```bash
gpg --keyserver hkps://keys.openpgp.org --send-keys KEY_ID     # Upload key
gpg --keyserver hkps://keys.openpgp.org --recv-keys KEY_ID     # Download key
gpg --keyserver hkps://keys.openpgp.org --search-keys "email"  # Search
```

---

## age Encryption

### Encrypt / Decrypt
```bash
# Recipient-based (asymmetric)
age -r age1publickey... -o file.age file.txt       # Encrypt
age -d -i key.txt -o file.txt file.age             # Decrypt
age -R recipients.txt -o file.age file.txt         # Multiple recipients from file

# Passphrase-based (symmetric)
age -p -o file.age file.txt                        # Encrypt with passphrase
age -d -o file.txt file.age                        # Decrypt (prompts for passphrase)
```

### Key Generation
```bash
age-keygen -o key.txt                              # Generate key pair
age-keygen                                         # Print to stdout
# Public key is shown in comment, private key in file
```

### Pipe and Stream
```bash
tar czf - dir/ | age -r age1pub... -o backup.tar.gz.age  # Encrypt archive
age -d -i key.txt backup.tar.gz.age | tar xzf -          # Decrypt and extract
```

---

## OpenSSL

### Random Generation
```bash
openssl rand -hex 32                               # 32 random hex bytes
openssl rand -base64 32                            # 32 random bytes as base64
openssl rand -out random.bin 256                   # 256 random bytes to file
```

### Symmetric Encryption
```bash
# Encrypt
openssl enc -aes-256-cbc -salt -pbkdf2 -iter 100000 -in file.txt -out file.enc
openssl enc -chacha20 -salt -pbkdf2 -in file.txt -out file.enc

# Decrypt
openssl enc -d -aes-256-cbc -pbkdf2 -iter 100000 -in file.enc -out file.txt

# With explicit password (avoid for security)
openssl enc -aes-256-cbc -salt -pbkdf2 -pass pass:mypassword -in file.txt -out file.enc
```

### RSA Key Operations
```bash
# Generate RSA key
openssl genrsa -out private.pem 4096              # Unencrypted
openssl genrsa -aes256 -out private.pem 4096      # Encrypted

# Extract public key
openssl rsa -in private.pem -pubout -out public.pem

# Encrypt/decrypt with RSA (small data only)
openssl rsautl -encrypt -pubin -inkey public.pem -in secret.txt -out secret.enc
openssl rsautl -decrypt -inkey private.pem -in secret.enc -out secret.txt
```

### Certificate Creation

#### Self-Signed Certificate
```bash
# One-liner (key + cert)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes \
  -subj "/CN=localhost"

# Ed25519
openssl req -x509 -newkey ed25519 -keyout key.pem -out cert.pem -days 365 -nodes \
  -subj "/CN=localhost"

# With Subject Alternative Names
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes \
  -subj "/CN=myapp.local" \
  -addext "subjectAltName=DNS:myapp.local,DNS:localhost,IP:127.0.0.1"
```

#### Certificate Signing Request (CSR)
```bash
# Generate CSR
openssl req -new -newkey rsa:4096 -keyout key.pem -out request.csr -nodes \
  -subj "/C=US/ST=State/L=City/O=Org/CN=example.com"

# Generate CSR from existing key
openssl req -new -key existing-key.pem -out request.csr

# View CSR
openssl req -in request.csr -text -noout -verify
```

#### Sign CSR with CA
```bash
openssl x509 -req -in request.csr -CA ca-cert.pem -CAkey ca-key.pem \
  -CAcreateserial -out signed-cert.pem -days 365 -sha256
```

### Certificate Inspection
```bash
openssl x509 -in cert.pem -text -noout             # Full certificate details
openssl x509 -in cert.pem -noout -dates            # Valid from/to dates
openssl x509 -in cert.pem -noout -subject          # Subject (CN, O, etc.)
openssl x509 -in cert.pem -noout -issuer           # Issuer
openssl x509 -in cert.pem -noout -fingerprint -sha256  # SHA-256 fingerprint
openssl x509 -in cert.pem -noout -serial           # Serial number
openssl x509 -in cert.pem -noout -purpose          # Allowed purposes
openssl x509 -in cert.pem -noout -ext subjectAltName  # SANs
openssl x509 -in cert.pem -noout -checkend 2592000 # Check if expires within 30 days
```

### TLS Debugging
```bash
# Basic connection test
openssl s_client -connect host:443 </dev/null 2>/dev/null

# With SNI (required for most modern servers)
openssl s_client -connect host:443 -servername host </dev/null 2>/dev/null

# Show full certificate chain
openssl s_client -connect host:443 -showcerts </dev/null 2>/dev/null

# Extract server certificate
openssl s_client -connect host:443 -servername host </dev/null 2>/dev/null | \
  openssl x509 -noout -subject -issuer -dates

# Test specific TLS version
openssl s_client -connect host:443 -tls1_2 </dev/null 2>/dev/null
openssl s_client -connect host:443 -tls1_3 </dev/null 2>/dev/null

# Show negotiated cipher
openssl s_client -connect host:443 </dev/null 2>/dev/null | grep 'Cipher is'

# List available ciphers
openssl ciphers -v 'HIGH:!aNULL:!MD5'
```

### Verify Certificate Chain
```bash
openssl verify -CAfile ca-bundle.pem cert.pem
openssl verify -CAfile ca.pem -untrusted intermediate.pem cert.pem
```

### PKCS#12 (PFX)
```bash
# Create .p12 from cert + key
openssl pkcs12 -export -out bundle.p12 -inkey key.pem -in cert.pem -certfile ca.pem

# Extract from .p12
openssl pkcs12 -in bundle.p12 -out cert.pem -clcerts -nokeys    # Cert only
openssl pkcs12 -in bundle.p12 -out key.pem -nocerts -nodes       # Key only
openssl pkcs12 -in bundle.p12 -out all.pem -nodes                # Everything
```

### DH Parameters
```bash
openssl dhparam -out dhparam.pem 2048              # Generate DH parameters
openssl dhparam -out dhparam.pem 4096              # Stronger (slower)
```

---

## mkcert (Local Development Certificates)

```bash
mkcert -install                                    # Install local CA
mkcert localhost                                   # Cert for localhost
mkcert localhost 127.0.0.1 ::1                    # Multiple names
mkcert "*.local.dev" localhost                     # Wildcard
mkcert -key-file key.pem -cert-file cert.pem localhost  # Custom filenames

# Show CA root location
mkcert -CAROOT
```

---

## certbot / Let's Encrypt

```bash
# Standalone (port 80 must be free)
sudo certbot certonly --standalone -d example.com

# Webroot (existing web server)
sudo certbot certonly --webroot -w /var/www/html -d example.com

# DNS challenge (wildcard certs)
sudo certbot certonly --manual --preferred-challenges dns -d "*.example.com"

# Renew
sudo certbot renew                                 # Renew all
sudo certbot renew --dry-run                       # Test renewal

# List certificates
sudo certbot certificates

# Revoke
sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem
```

---

## Password Hashing

```bash
# SHA-512 system password hash
openssl passwd -6                                  # Interactive
openssl passwd -6 -salt "$(openssl rand -hex 8)" "password"  # With salt

# Apache htpasswd
htpasswd -c .htpasswd user                         # Create file, add user
htpasswd .htpasswd newuser                         # Add user to existing
htpasswd -B .htpasswd user                         # bcrypt hash
htpasswd -nb user password                         # Print to stdout (no file)

# mkpasswd (from whois package)
mkpasswd -m sha-512                                # SHA-512
mkpasswd -m bcrypt                                 # bcrypt

# Python hashlib
python3 -c "import hashlib; print(hashlib.sha256(b'text').hexdigest())"
```

---

## sops (Encrypted Config Files)

```bash
# Encrypt file
sops --encrypt --age age1publickey... config.yaml > config.enc.yaml
sops --encrypt --in-place config.yaml              # In-place

# Decrypt
sops --decrypt config.enc.yaml                     # To stdout
sops --decrypt --in-place config.enc.yaml          # In-place

# Edit (decrypt, edit, re-encrypt)
sops config.enc.yaml                               # Opens in $EDITOR

# With .sops.yaml config
# .sops.yaml:
# creation_rules:
#   - path_regex: '.*\.enc\.yaml$'
#     age: 'age1publickey...'
sops --encrypt config.yaml > config.enc.yaml       # Uses .sops.yaml rules
```

---

## pass (Password Store)

```bash
pass init "GPG-KEY-ID"                             # Initialize store
pass insert email/gmail                            # Add password
pass email/gmail                                   # Show password
pass -c email/gmail                                # Copy to clipboard
pass generate email/newaccount 20                  # Generate 20-char password
pass edit email/gmail                              # Edit entry
pass find gmail                                    # Search
pass ls                                            # List all entries
pass rm email/old                                  # Remove entry
pass git push                                      # Sync to git remote
```

---

## Common Workflows

### Verify Downloaded File
```bash
# Download file and checksum
curl -LO https://example.com/file.tar.gz
curl -LO https://example.com/SHA256SUMS
sha256sum -c SHA256SUMS --ignore-missing
```

### Encrypt File for Email
```bash
# GPG (recipient must have GPG)
gpg --armor -e -r "recipient@email.com" sensitive.pdf

# age (simpler, recipient needs age)
age -r age1recipientpublickey... -o sensitive.pdf.age sensitive.pdf
```

### Generate and Deploy SSL Certificate
```bash
# Development (mkcert)
mkcert -install && mkcert localhost 127.0.0.1

# Production (Let's Encrypt)
sudo certbot certonly --standalone -d mysite.com
# Certs at: /etc/letsencrypt/live/mysite.com/{fullchain,privkey}.pem
```

### Rotate SSH Keys
```bash
# Generate new key
ssh-keygen -t ed25519 -f ~/.ssh/new_key -C "rotated $(date +%Y-%m-%d)"

# Deploy to servers
ssh-copy-id -i ~/.ssh/new_key.pub user@server1
ssh-copy-id -i ~/.ssh/new_key.pub user@server2

# Test new key
ssh -i ~/.ssh/new_key user@server1 "echo ok"

# Remove old key from servers
ssh user@server1 "sed -i '/OLD_KEY_COMMENT/d' ~/.ssh/authorized_keys"
```

### Check Certificate Expiry
```bash
# Remote server
echo | openssl s_client -servername host -connect host:443 2>/dev/null | \
  openssl x509 -noout -dates

# Local cert file
openssl x509 -in cert.pem -noout -enddate

# Expires within 30 days?
openssl x509 -in cert.pem -noout -checkend 2592000 && echo "OK" || echo "EXPIRING"
```

### JWT (JSON Web Token) Operations

```bash
# Decode JWT payload (no verification)
echo "$JWT" | cut -d. -f2 | base64 -d 2>/dev/null | jq .

# Decode JWT header
echo "$JWT" | cut -d. -f1 | base64 -d 2>/dev/null | jq .

# Check JWT expiry
echo "$JWT" | cut -d. -f2 | base64 -d 2>/dev/null | jq '.exp | todate'

# Create JWT with openssl (HS256)
HEADER=$(echo -n '{"alg":"HS256","typ":"JWT"}' | base64 -w0 | tr '+/' '-_' | tr -d '=')
PAYLOAD=$(echo -n '{"sub":"user","iat":'$(date +%s)'}' | base64 -w0 | tr '+/' '-_' | tr -d '=')
SIG=$(echo -n "$HEADER.$PAYLOAD" | openssl dgst -sha256 -hmac "secret" -binary | base64 -w0 | tr '+/' '-_' | tr -d '=')
echo "$HEADER.$PAYLOAD.$SIG"

# Verify JWT signature (HS256)
EXPECTED=$(echo -n "$HEADER.$PAYLOAD" | openssl dgst -sha256 -hmac "secret" -binary | base64 -w0 | tr '+/' '-_' | tr -d '=')
[ "$SIG" = "$EXPECTED" ] && echo "Valid" || echo "Invalid"
```

### HMAC

```bash
# Generate HMAC
echo -n "message" | openssl dgst -sha256 -hmac "secret-key"
openssl dgst -sha256 -hmac "secret-key" file.txt

# HMAC with hex key
echo -n "message" | openssl dgst -sha256 -mac HMAC -macopt hexkey:aabbccdd

# Webhook signature verification pattern
PAYLOAD='{"event":"push"}'
EXPECTED_SIG="sha256=$(echo -n "$PAYLOAD" | openssl dgst -sha256 -hmac "$WEBHOOK_SECRET" | cut -d' ' -f2)"
```

### Password Generation

```bash
# OpenSSL random
openssl rand -base64 32 | tr -d '/+=' | head -c 24   # 24-char alphanumeric

# /dev/urandom method
< /dev/urandom tr -dc 'A-Za-z0-9!@#$%' | head -c 32  # 32-char with symbols

# pwgen
pwgen 16 1                                     # One 16-char password
pwgen -s 24 1                                  # Secure (fully random)
pwgen -sy 20 5                                 # 5 passwords with symbols
```

### Certificate Key Matching

```bash
# Verify cert and key match (compare modulus hashes)
openssl x509 -noout -modulus -in cert.pem | md5sum
openssl rsa -noout -modulus -in key.pem | md5sum
# If hashes match, cert and key are a pair

# Also check CSR matches
openssl req -noout -modulus -in request.csr | md5sum

# One-liner match check
diff <(openssl x509 -noout -modulus -in cert.pem) <(openssl rsa -noout -modulus -in key.pem) && echo "MATCH" || echo "MISMATCH"
```

### Combined Workflows

```bash
# JWT decode pipeline — extract token from API response and decode
curl -s http://localhost:3000/auth/login -d '{"user":"admin","pass":"secret"}' | \
  jq -r '.access_token' | cut -d. -f2 | base64 -d 2>/dev/null | jq .

# Cert expiry monitoring across hosts
for host in host1.example.com host2.example.com; do
  echo -n "$host: "
  echo | openssl s_client -servername "$host" -connect "$host:443" 2>/dev/null | \
    openssl x509 -noout -enddate
done

# Encrypted cloud backup pipeline
tar czf - /sensitive | gpg -c --batch --passphrase-file pw.txt | \
  rclone rcat remote:backup/$(date +%Y%m%d).tar.gz.gpg
```

---

## Installation

```bash
# Core tools (usually pre-installed)
sudo apt install coreutils openssl gnupg2

# age encryption
# https://github.com/FiloSottile/age/releases
# or: sudo apt install age

# mkcert (local dev certs)
# https://github.com/FiloSottile/mkcert/releases

# certbot (Let's Encrypt)
sudo apt install certbot

# sops (encrypted config)
# https://github.com/getsops/sops/releases

# pass (password store)
sudo apt install pass

# rhash (multi-algorithm hashing)
sudo apt install rhash

# htpasswd
sudo apt install apache2-utils

# mkpasswd
sudo apt install whois
```
