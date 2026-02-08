---
description: "Cryptography and security - file corrupted need to verify, verify download integrity, certificate expired, SSL handshake failed, need to encrypt sensitive data, hashing, checksums, and key management (openssl, gpg, ssh-keygen, age)"
when_to_use: "Use when: need to verify a download wasn't corrupted, file integrity check needed, certificate expired or SSL error, SSL handshake failed, need to encrypt files for transfer, generate SSH keys, manage GPG keys, create certificates, hash passwords, TLS connection debugging. Triggers: file corrupted, verify download, certificate expired, SSL handshake failed, need to encrypt, SSL error, HTTPS not working, certificate invalid, hash, md5, sha256, checksum, base64, encrypt, decrypt, gpg, ssh key, certificate, openssl, sign, verify, tls, cert, age, sops, password hash, ssl certificate, jwt decode, jwt token, hmac, password generate, key pair, verify signature, certificate expiry, pfx, pem."
when-to-use: "Use when: need to verify a download wasn't corrupted, file integrity check needed, certificate expired or SSL error, SSL handshake failed, need to encrypt files for transfer, generate SSH keys, manage GPG keys, create certificates, hash passwords, TLS connection debugging. Triggers: file corrupted, verify download, certificate expired, SSL handshake failed, need to encrypt, SSL error, HTTPS not working, certificate invalid, hash, md5, sha256, checksum, base64, encrypt, decrypt, gpg, ssh key, certificate, openssl, sign, verify, tls, cert, age, sops, password hash, ssl certificate, jwt decode, jwt token, hmac, password generate, key pair, verify signature, certificate expiry, pfx, pem."
argument-hint: "[describe the cryptography task]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Cryptography Task Router

You have access to the **full conversation context**. Your job is to analyze the user's cryptography need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Cryptography requests span hashing, encoding, encryption, key management, and certificates. Analyze the conversation to identify:

### 1. Goal
What crypto operation is needed?
- Hash / checksum (verify file integrity, compute digest)
- Encode / decode (base64, hex, URL-safe)
- Encrypt / decrypt (symmetric, asymmetric, file, stream)
- Sign / verify (GPG signatures, code signing)
- Key generation (SSH, GPG, RSA, Ed25519, age)
- Certificate operations (create, inspect, self-signed, CSR, CA)
- TLS debugging (test connection, cipher info, cert chain)
- Password hashing (htpasswd, system passwords)
- Secret management (sops, pass, vault)

### 2. Target
What data or file is involved?
- File path(s) mentioned in conversation
- String / stdin data
- Remote host (for TLS debugging)
- Existing key or cert paths

### 3. Algorithm / Format Preference
- Hash: SHA-256, SHA-512, MD5, BLAKE2
- Key type: Ed25519, RSA-4096, ECDSA
- Encryption: AES-256-CBC, ChaCha20, age
- Certificate: X.509, PEM, DER, PKCS#12

### 4. Context
- Verifying a download? (checksum)
- Encrypting for email/transfer? (GPG, age)
- Setting up SSH access? (keygen, copy-id)
- Deploying TLS? (cert creation, chain verification)
- Managing secrets in config? (sops, pass)

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| File checksum | sha256sum, b2sum | Verify integrity |
| Base64 encode/decode | base64 | Also: basenc, xxd |
| Symmetric encrypt | gpg -c, openssl enc, age -p | Password-based |
| Asymmetric encrypt | gpg -e -r, age -r | Recipient-based |
| SSH key management | ssh-keygen, ssh-agent | Ed25519 recommended |
| GPG sign/verify | gpg --sign, --verify | Detached or inline |
| Certificate creation | openssl req, mkcert | Self-signed or CSR |
| Certificate inspection | openssl x509 | Dates, issuer, chain |
| TLS debugging | openssl s_client | Cipher, protocol, SNI |
| Password hashing | openssl passwd, htpasswd | System or web |
| Secret management | sops, pass | Encrypted config files |

---

## Argument Format

Construct a structured argument string:

```
<goal> on <target> - algorithm: <pref>, format: <pref>, context: <details>
```

### Examples

```
compute sha256 checksum of /tmp/download.iso - verify against expected hash abc123
```
```
generate SSH key for deploy@production - type: ed25519, no passphrase, output: ~/.ssh/deploy_key
```
```
encrypt config.json with age - recipient public key: age1abc..., output: config.json.age
```
```
inspect TLS certificate on api.example.com:443 - show expiry, issuer, and chain
```
```
create self-signed certificate for localhost - valid 365 days, include SANs for 127.0.0.1
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool to invoke `task-crypto-recipes` with your constructed argument.

### When to Ask Instead

**Ask the user if:**
- Encryption method not specified and multiple options apply
- Target file path cannot be determined
- Key management operation could have security implications

**Use Read tool first if:**
- Need to check if key/cert file exists
- Need to inspect existing key type before generating new one
