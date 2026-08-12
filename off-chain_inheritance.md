# Nunchuk Off-Chain Inheritance

## Hardware Wallet Integration Specification

**Status:** Draft  
**Doc version:** 0.2  
**Audience:** Hardware wallet / signer manufacturers and engineering teams

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

---

## 1. Purpose

Nunchuk supports inheritance for multisig Bitcoin wallets via a designated **Inheritance Key**. Nunchuk MUST NOT receive the plaintext private key.

Instead, the hardware wallet produces an **on-device encrypted backup**. Nunchuk stores only the ciphertext. The **Backup Password** stays outside Nunchuk and is delivered to the beneficiary separately.

This document specifies the hardware-wallet behavior needed to integrate with that flow.

---

## 2. Motivation and Security Model

Example wallet:

```text
2-of-4 Multisig

Key A ─ Owner
Key B ─ Owner
Key C ─ Inheritance Key
Key D ─ Nunchuk Platform Key
```

Normal spending does not need the Inheritance Key. On inheritance, the beneficiary needs it (together with the Platform Key) to satisfy the multisig — but may not have the original hardware device. A recoverable backup is therefore required.

If Nunchuk ever handled the plaintext key, a Nunchuk compromise would put every inheritance plan at risk. Encryption MUST happen on the hardware wallet:

```text
Hardware Wallet ──encrypt locally──► Encrypted Backup ──► Nunchuk
Backup Password ─────────────────────────────────────► Beneficiary
```

### Responsibilities

| Party | Responsibility |
| --- | --- |
| **Hardware wallet** | Generate/import Inheritance Key; generate Backup Password; encrypt backup; never export plaintext during backup creation |
| **Nunchuk** | Store ciphertext; bind it to a plan; gate claims; guide the beneficiary; co-sign with the Platform Key when eligible |
| **Beneficiary** | Obtain Backup Password; claim; decrypt locally; sign with the recovered Inheritance Key |

### Security boundary

Nunchuk MUST NOT receive: Inheritance private key, seed containing that key, Backup Password, encryption key, or plaintext backup.

Nunchuk SHOULD receive only: encrypted backup, backup metadata, and public key / fingerprint.

> Even if Nunchuk's inheritance-backup database is fully compromised, the Inheritance Key remains unrecoverable without the separately held Backup Password.

---

## 3. User Flows

### 3.1 Owner setup

```text
Owner → creates plan in Nunchuk → selects HW as Inheritance Key
         ↓
Hardware Wallet → holds Inheritance Key
                → generates Backup Password
                → creates encrypted backup
         ↓
Encrypted Backup → Nunchuk (ciphertext only)

Backup Password → Beneficiary / Trustee (out of band)
```

### 3.2 Claim

```text
Beneficiary → Magic Phrase / inheritance identifier → Nunchuk
Nunchuk → verifies eligibility → returns Encrypted Backup
Beneficiary → decrypts locally with Backup Password
            → recovers Inheritance Key → signs TX
Nunchuk Platform Key co-signs → multisig satisfied → Bitcoin
```

The Backup Password MUST NOT be sent to Nunchuk at any point.

---

## 4. Required Hardware Capability

A compatible device MUST expose the equivalent of:

```text
create_inheritance_backup()
```

This MUST run in the trusted hardware environment: generate a random Backup Password, encrypt the Inheritance Key backup on-device, and return only ciphertext (plus metadata) to the host. The private key MUST NOT be exported as part of this operation.

---

## 5. Backup Password Requirements

The hardware wallet SHOULD generate the Backup Password with its hardware RNG / secure random source.

The password SHOULD be:

- randomly generated;
- independent of the wallet seed, device PIN, and Inheritance Key;
- high enough entropy to resist offline guessing.

### User-friendly format (strongly recommended)

Heirs are often non-technical and may need to handle the Backup Password years later. The Backup Password SHOULD therefore use a **human-friendly encoding**, ideally a **word mnemonic similar to BIP39** (e.g. a random 12-word phrase), so it is easy to write down, read aloud, and enter without transcription errors.

The password need not be a BIP39 *seed* for a wallet — only a BIP39-*like* word encoding of high-entropy material is required.

COLDCARD's random 12-word on-device Backup Password is the reference pattern. See [References](#references).

Vendors MAY use another representation if it has sufficient entropy, can be reliably recorded and entered by a non-technical beneficiary, and is never sent to Nunchuk.

---

## 6. Encrypted Backup Requirements

The backup MUST be generated **locally** on the device and MUST provide:

| Property | Requirement |
| --- | --- |
| **Confidentiality** | Without the Backup Password, the private key cannot be recovered |
| **Integrity** | Tampering or corruption MUST be detectable |
| **Portability** | SHOULD NOT require the original device once the Backup Password is known |
| **Versioning** | Format MUST identify its version |
| **Deterministic recovery** | Same ciphertext + password → same Inheritance Key |

### Contents

At minimum, enough to reconstruct the Inheritance Key:

```text
Private key / seed material
Key type
Key origin / derivation information, if applicable
```

Additional device metadata is allowed. The internal format is vendor-specific (e.g. COLDCARD uses encrypted 7z / AES-256-CBC). Nunchuk requires the **behavior**, not a shared file format.

---

## 7. Interface

### Create backup

```text
create_inheritance_backup() → encrypted_backup, backup_metadata
```

The Backup Password is shown to the owner on the device (or equivalent secure channel) and MUST NOT be returned to Nunchuk.

### Backup metadata

Recommended fields:

```text
backup_version
key_type
key_fingerprint
public_key
derivation_path / key origin
```

Example logical payload:

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

Serialization is implementation-specific.

### Key identity

The device SHOULD expose a stable fingerprint and preferably the public key so Nunchuk can bind the backup to the registered Inheritance Key. After recovery, the key MUST match that registration.

---

## 8. Critical Security Rule

**Not acceptable:** export plaintext private key to Nunchuk for Nunchuk to encrypt.

**Required:** encrypt inside hardware; only ciphertext leaves the device; Backup Password never goes to Nunchuk.

```text
Hardware (Private Key + Backup Password) → Encrypted Backup → Nunchuk
Beneficiary decrypts locally with Backup Password → Inheritance Key
```

---

## 9. UX and Verification

The device UI SHOULD state clearly that encryption and password generation happen on-device, and that Nunchuk receives only the ciphertext. The owner SHOULD confirm they have recorded the Backup Password before finishing.

The device SHOULD support backup verification (re-enter password, decrypt/check internally, confirm validity) so failures are found at setup time, not years later during a claim.

---

## 10. Recovery Models

| Model | Description |
| --- | --- |
| **A — Software** | Beneficiary decrypts with Backup Password on a software wallet (simplest for inheritance) |
| **B — Hardware** | Beneficiary restores into a compatible hardware device (stronger isolation) |

Formats SHOULD support both where practical; this is vendor-specific.

---

## 11. Requirements Checklist

### Required

1. On-device encrypted backup generation  
2. Hardware-generated random Backup Password  
3. Encrypted backup export  
4. No plaintext private-key export during backup creation  
5. Recovery using Backup Password  
6. Stable key fingerprint / public-key identification  
7. Versioned backup format  
8. Integrity / authentication protection  
9. Documented backup / recovery API or integration path  

### Strongly recommended

10. Backup verification  
11. User-friendly Backup Password (BIP39-like mnemonic) for non-technical heirs  
12. Portable recovery without the original device  
13. Documented cryptographic construction  
14. Backward compatibility / version migration  

### Out of scope for the hardware wallet

Inheritance policy, beneficiary identity, activation date, Nunchuk servers, Platform Key custody, inheritance transaction construction, claim status, and retaining the Backup Password after creation.

Hardware responsibility in one line: **securely create and recover an encrypted backup of the Inheritance Key.**

---

## 12. Glossary

| Term | Meaning |
| --- | --- |
| **Inheritance Key** | Multisig cosigner reserved for the inheritance path |
| **Backup Password** | On-device high-entropy secret that decrypts the backup; never sent to Nunchuk |
| **Encrypted Backup** | On-device ciphertext needed to recover the Inheritance Key |
| **Platform Key** | Nunchuk cosigner used when a claim is eligible |
| **Magic Phrase** | Identifier used to locate / start a claim in Nunchuk |
| **Trustee** | Optional holder/conveyor of the Backup Password for the beneficiary |
| **Owner** | Creates the plan and registers the Inheritance Key |
| **Beneficiary** | Claims and spends after activation conditions are met |

---

## References

- [COLDCARD encrypted backups](https://coldcard.com/docs/backups/)
- [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119)
- [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
