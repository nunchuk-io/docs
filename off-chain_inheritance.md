# Nunchuk Off-Chain Inheritance

## Hardware Wallet Integration Specification

**Status:** Draft  
**Doc version:** 0.1  
**Audience:** Hardware wallet / signer manufacturers and engineering teams

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

---

## 1. Purpose

Nunchuk provides an inheritance mechanism for multisig Bitcoin wallets.

For the inheritance path, Nunchuk needs the beneficiary to eventually recover a designated **Inheritance Key**. However, Nunchuk must never have access to the plaintext private key.

We therefore use a **hardware-generated encrypted backup**:

> The hardware wallet encrypts the Inheritance Key backup locally. Nunchuk stores only the encrypted backup. The Backup Password remains outside Nunchuk and is delivered to the beneficiary separately.

This document defines the hardware-wallet capability required to integrate with this inheritance flow.

---

## 2. Why an Encrypted Backup Is Needed

A normal multisig wallet may have several signing keys:

```text
2-of-4 Multisig

Key A ─ Owner
Key B ─ Owner
Key C ─ Inheritance Key
Key D ─ Nunchuk Platform Key
```

During normal operation, the Inheritance Key is not required.

When inheritance is triggered, the beneficiary needs access to the Inheritance Key:

```text
Inheritance Key + Platform Key
             ↓
       satisfy multisig
             ↓
       spend the wallet
```

The problem is that the beneficiary may not physically possess the hardware device that holds the Inheritance Key.

We therefore need a recoverable backup of that key.

However:

### Nunchuk must not receive the private key.

If Nunchuk received the plaintext private key, compromising Nunchuk's infrastructure could compromise every inheritance plan using the service.

Instead:

```text
Hardware Wallet
      │
      │ encrypt locally
      ▼
Encrypted Backup
      │
      ▼
   Nunchuk
```

The decryption secret remains separate:

```text
Backup Password
      │
      ▼
Beneficiary
```

Therefore, a database compromise at Nunchuk does not reveal the inheritance private keys.

---

## 3. Trust Model

The hardware wallet is the trusted environment for generating and protecting the private key.

### Hardware wallet is responsible for

- generating/importing the Inheritance Key;
- generating the Backup Password;
- encrypting the backup;
- ensuring the plaintext private key does not leave the device during backup creation.

### Nunchuk is responsible for

- storing the encrypted backup;
- associating the backup with an inheritance plan;
- determining when an inheritance claim becomes eligible;
- guiding the beneficiary through the claim process;
- providing the Platform Key signature when policy conditions are satisfied.

### Beneficiary is responsible for

- obtaining the Backup Password;
- initiating the inheritance claim;
- decrypting/recovering the backup;
- using the recovered Inheritance Key to sign the transaction.

---

## 4. Security Boundary

The following information MUST NOT be provided to Nunchuk:

```text
Inheritance private key
Seed phrase containing the Inheritance Key
Backup Password
Encryption key
Plaintext backup
```

Nunchuk SHOULD receive only:

```text
Encrypted backup
Backup metadata
Public key / fingerprint information
```

The intended security property is:

> Even if Nunchuk's entire inheritance-backup database is compromised, an attacker cannot recover the Inheritance Key without the separately held Backup Password.

---

## 5. High-Level User Flow

### 5.1 Owner Setup

```text
Owner
  │
  ├── creates inheritance plan in Nunchuk
  │
  ├── selects hardware wallet as Inheritance Key
  │
  ▼
Hardware Wallet
  │
  ├── generates/holds Inheritance Key
  │
  ├── generates random Backup Password
  │
  ├── creates encrypted backup
  │
  ▼
Encrypted Backup
  │
  └──────────────► Nunchuk
                    │
                    └── stores ciphertext
```

The owner separately stores/distributes:

```text
Backup Password
      │
      └──► Beneficiary / Trustee
```

### 5.2 Claim Flow

After the inheritance activation condition is satisfied:

```text
Beneficiary
    │
    ├── Magic Phrase / inheritance identifier
    │
    ▼
Nunchuk
    │
    ├── finds inheritance plan
    ├── verifies claim eligibility
    │
    ▼
Encrypted Backup
    │
    └── downloaded by beneficiary
             │
             │ Backup Password
             ▼
        Decrypt locally
             │
             ▼
       Inheritance Key
             │
             ▼
       Sign inheritance TX
             │
             +
        Nunchuk Platform Key
             │
             ▼
       Multisig satisfied
             │
             ▼
          Bitcoin
```

The Backup Password is **never sent to Nunchuk**.

---

## 6. Required Hardware Wallet Capability

A compatible hardware wallet MUST provide a mechanism equivalent to:

```text
create_inheritance_backup()
```

The operation MUST happen inside the trusted hardware environment.

Conceptually:

```text
Hardware Wallet

Private Key
     │
     ├── generate random Backup Password
     │
     └── encrypt backup
             │
             ▼
      Encrypted Backup
```

The device then exposes the encrypted backup to the host application.

The private key itself MUST NOT be exported as part of this operation.

---

## 7. Backup Password Requirements

The hardware wallet SHOULD generate the Backup Password using its hardware RNG / secure random source.

The password SHOULD be:

- randomly generated;
- independent of the wallet seed;
- independent of the device PIN;
- independent of the Inheritance Key;
- sufficiently high entropy to resist offline guessing.

COLDCARD's implementation is the reference example: it generates a random 12-word Backup Password on-device and uses it to protect the encrypted backup. See [References](#references).

The hardware vendor MAY use a different password representation, provided that:

1. it has sufficient entropy;
2. the user can reliably record it;
3. the beneficiary can use it for recovery;
4. Nunchuk never receives it.

---

## 8. Encrypted Backup Requirements

The hardware wallet MUST generate the encrypted backup **locally**.

The backup MUST provide:

### Confidentiality

Without the Backup Password, the private key cannot be recovered.

### Integrity

Modification/corruption of the encrypted backup MUST be detectable.

### Portability

The backup SHOULD NOT require the original hardware device to decrypt once the beneficiary has the Backup Password.

### Versioning

The format MUST identify its version so future implementations can support migrations.

### Deterministic recovery

Given:

```text
Encrypted Backup
+
Backup Password
```

the beneficiary SHOULD be able to recover the same Inheritance Key.

---

## 9. Backup Contents

The encrypted backup SHOULD contain sufficient information to reconstruct the Inheritance Key.

At minimum:

```text
Private key / seed material
Key type
Key origin / derivation information, if applicable
```

The backup MAY contain additional device-specific metadata.

The exact backup format is hardware-vendor-specific.

Nunchuk does **not** require hardware vendors to use the same internal backup format.

For example, COLDCARD currently uses an encrypted 7z backup with AES-256-CBC. See [References](#references).

The important interoperability requirement is the behavior, not necessarily the file format.

---

## 10. Required Interface Between Hardware Wallet and Nunchuk

The integration SHOULD expose the following logical information.

### Create backup

```text
create_inheritance_backup()
```

Returns:

```text
encrypted_backup
backup_metadata
```

The Backup Password is displayed/provided to the owner separately and MUST NOT be returned to Nunchuk.

### Backup metadata

Recommended metadata:

```text
backup_version
key_type
key_fingerprint
public_key
derivation_path / key origin
```

Example (logical payload returned to Nunchuk; fields may be nested or flattened):

```json
{
  "backup_version": 1,
  "key_type": "secp256k1",
  "key_fingerprint": "ABCD1234",
  "public_key": "...",
  "derivation_path": "m/48'/0'/0'/2'",
  "encrypted_backup": "..."
}
```

The exact serialization is implementation-specific.

---

## 11. Critical Security Requirements

The following flow is **NOT acceptable**:

```text
Hardware
   │
   ▼
Plaintext Private Key
   │
   ▼
Nunchuk
   │
   ▼
Nunchuk encrypts it
```

The following flow is required:

```text
Hardware
   │
   ├── Private Key
   │
   ├── Backup Password
   │
   ▼
Encrypt inside hardware
   │
   ▼
Encrypted Backup
   │
   ▼
Nunchuk
```

This distinction is fundamental to the protocol.

The following MUST NOT occur at any point:

```text
Backup Password → Nunchuk
```

The complete claim can be performed as:

```text
Nunchuk
  │
  ├── provides encrypted backup
  │
  ▼
Beneficiary device
  │
  ├── receives Backup Password
  │
  ├── decrypts backup locally
  │
  ▼
Inheritance Key
```

This means Nunchuk can provide cloud storage for the encrypted backup without becoming a custodian of the Inheritance Key.

---

## 12. User Experience Requirements

The hardware-wallet UI SHOULD make the security boundary clear.

Example:

```text
Create Inheritance Backup?

This backup allows your beneficiary to recover
your Inheritance Key.

The backup will be encrypted on this device.

Your Backup Password will be generated on this device.

Nunchuk will receive only the encrypted backup.

[Continue]
```

Then:

```text
BACKUP PASSWORD

word1 word2 word3 ...
...

Write these words down.

This password is required to recover
your Inheritance Key.

Nunchuk will never receive this password.
```

The user SHOULD be required to confirm the password has been recorded.

---

## 13. Backup Verification

The hardware device SHOULD provide a way to verify that the encrypted backup can be recovered.

For example:

```text
Verify Backup
      │
      ▼
Enter / confirm Backup Password
      │
      ▼
Decrypt / verify internally
      │
      ▼
✓ Backup is valid
```

This reduces the risk of a beneficiary discovering years later that the backup was never correctly created.

COLDCARD provides backup verification functionality as part of its encrypted-backup workflow. See [References](#references).

---

## 14. Key Identity Verification

Nunchuk needs to know which key the encrypted backup represents.

The hardware wallet SHOULD expose a stable identifier such as:

```text
key fingerprint
```

and preferably:

```text
public key
```

Nunchuk uses this to verify:

```text
Encrypted Backup
        ↓
Recovered Key
        ↓
Expected Inheritance Key
```

The recovered key MUST match the key originally registered as the Inheritance Key.

This prevents accidental use of the wrong backup.

---

## 15. Recovery Flow on Hardware / Software

There are two possible models.

### Model A — Software recovery

The beneficiary decrypts the backup using the Backup Password and obtains the Inheritance Key on a software wallet.

This is the simplest model for inheritance.

### Model B — Hardware recovery

The beneficiary imports/restores the backup into a compatible hardware device.

This provides stronger key isolation.

The backup format SHOULD preferably support both models, although this is ultimately hardware-vendor-specific.

---

## 16. Requirements Checklist

### Required

1. **On-device encrypted backup generation**
2. **Hardware-generated random Backup Password**
3. **Encrypted backup export**
4. **No plaintext private-key export during backup creation**
5. **Recovery using Backup Password**
6. **Stable key fingerprint/public-key identification**
7. **Versioned backup format**
8. **Integrity/authentication protection**
9. **Documented backup/recovery API or integration mechanism**

### Strongly recommended

10. Backup verification
11. Human-readable Backup Password
12. Portable recovery format
13. Ability to recover without the original device
14. Documentation of the cryptographic construction
15. Backward compatibility/version migration

### Out of scope for the hardware wallet

The hardware wallet does NOT need to:

- implement Nunchuk's inheritance policy;
- know the beneficiary;
- know the activation date;
- communicate with the Nunchuk inheritance server;
- hold the Nunchuk Platform Key;
- perform the inheritance transaction;
- know whether a claim has been initiated;
- know the Backup Password after backup creation.

The hardware wallet's responsibility is simply:

> **Securely create and recover an encrypted backup of the designated Inheritance Key.**

Everything related to inheritance orchestration happens outside the hardware wallet.

---

## 17. Why This Architecture

This separation gives each component a clear responsibility:

```text
┌──────────────────────┐
│   Hardware Wallet    │
│                      │
│ Private-key custody  │
│ Backup encryption    │
│ Backup Password      │
└──────────┬───────────┘
           │
           │ encrypted backup
           ▼
┌──────────────────────┐
│       Nunchuk        │
│                      │
│ Backup storage       │
│ Inheritance policy   │
│ Claim orchestration  │
│ Platform signature   │
└──────────┬───────────┘
           │
           │ encrypted backup
           ▼
┌──────────────────────┐
│     Beneficiary      │
│                      │
│ Backup Password      │
│ Backup decryption    │
│ Inheritance signing  │
└──────────────────────┘
```

No single component needs to possess all the information required to compromise the inheritance path.

---

## 18. Reference Integration

COLDCARD's encrypted backup functionality is the reference implementation for this integration model.

The important properties we want to reproduce are:

```text
1. Backup Password generated by hardware
2. Backup encrypted by hardware
3. Plaintext key never sent to Nunchuk
4. Encrypted backup can be exported
5. Password remains separate from backup
6. Backup can later be decrypted/recovered
```

The exact cryptographic/container format does not have to be identical between hardware vendors.

Nunchuk's protocol SHOULD therefore define the **security and behavioral requirements**, while allowing each hardware vendor to retain its own secure backup format.

---

## 19. Glossary

| Term | Meaning |
| --- | --- |
| **Inheritance Key** | The designated multisig cosigner key reserved for the inheritance path. |
| **Backup Password** | High-entropy secret generated on the hardware wallet; used to decrypt the encrypted backup. Never sent to Nunchuk. |
| **Encrypted Backup** | Ciphertext produced on-device that contains material needed to recover the Inheritance Key. |
| **Platform Key** | Nunchuk-controlled cosigner key used to help satisfy the multisig policy when a claim is eligible. |
| **Magic Phrase** | Inheritance identifier the beneficiary uses to locate and start an inheritance claim in Nunchuk. |
| **Trustee** | Optional party who may hold or convey the Backup Password on behalf of the beneficiary. |
| **Owner** | Party who creates the inheritance plan and registers the Inheritance Key. |
| **Beneficiary** | Party entitled to claim and spend via the inheritance path after activation conditions are met. |

---

## 20. Summary

### User experience

```text
Owner
  ↓
Create inheritance plan
  ↓
Create encrypted backup on hardware
  ↓
Upload encrypted backup to Nunchuk
  ↓
Give Backup Password to beneficiary
  ↓
[time / inheritance condition]
  ↓
Beneficiary starts claim
  ↓
Downloads encrypted backup
  ↓
Decrypts locally with Backup Password
  ↓
Recovers Inheritance Key
  ↓
Signs transaction
  ↓
Nunchuk provides Platform signature
  ↓
Bitcoin transaction
```

### Hardware wallet requirement

> **Provide a secure, on-device encrypted backup mechanism for the designated Inheritance Key, with a hardware-generated high-entropy Backup Password that is never exposed to Nunchuk.**

### Core security property

```text
Nunchuk has:
    ✓ encrypted backup
    ✓ public key / fingerprint
    ✓ inheritance metadata

Nunchuk does NOT have:
    ✗ private key
    ✗ seed
    ✗ Backup Password
    ✗ encryption key
```

This is the required foundation for integrating a hardware wallet into Nunchuk's off-chain inheritance protocol.

---

## References

- [COLDCARD encrypted backups](https://coldcard.com/docs/backups/)
- [RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119)
