# E2EE for multi-party multisig in Bitcoin
This document specifies how multi-party multisig in Bitcoin can be facilitated using [Matrix](https://matrix.org/). 

Matrix is an open standard and communication protocol for real-time communication. Matrix supports end-to-end-encryption (E2EE) for one-on-one and group chats, thanks to its [Olm and Megolm protocols](https://matrix.org/docs/guides/end-to-end-encryption-implementation-guide).

Matrix clients that follow the below schema can construct Bitcoin transactions across different Matrix federations.

The schema is currently used in all [Nunchuk applications](https://nunchuk.io).

## API
See https://github.com/nunchuk-io/libnunchuk/blob/main/include/nunchukmatrix.h.

## Data structures
- Bitcoin wallet configurations are stored in the [BSMS format](https://github.com/bitcoin/bips/blob/master/bip-0129.mediawiki) (which is built on top of [Output Descriptors](https://bitcoinops.org/en/topics/output-script-descriptors/)). 
- Bitcoin transactions are stored in the [PSBT format](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki).
- PSBTs, in turn, are embedded within the [Matrix custom event types](https://spec.matrix.org/v1.4/client-server-api/#types-of-room-events) defined below.

## Workflows
There are 2 main flows:
- Wallet creation (or recovery)
  - Initialize wallet creation/recovery session
  - Join wallet
  - Leave/cancel wallet
  - Finalize wallet

- Transaction creation
  - Initialize transaction
  - Cancel or Sign transaction (repeated until enough signatures have been collected)
  - Broadcast transaction 

## Matrix notes
- If the Matrix room is E2EE-enabled, all Bitcoin wallet and transaction events will also be end-to-end-encrypted.
- [`m.relates_to`](https://spec.matrix.org/v1.4/client-server-api/#forming-relationships-between-events) is used to establish the relationship graph of events for each workflow.
- Matrix limits the event size to be [<= 64kB (65536 bytes)](https://spec.matrix.org/v1.4/client-server-api/#size-limits). This works for the majority of PSBTs, but not all.
- For extremely large PSBTs (such as Bitcoin transactions with lots of inputs and/or outputs), the PSBTs need to be converted to media files and [upload to the Matrix room](https://spec.matrix.org/unstable/client-server-api/#post_matrixmediav3upload).

## Wallet creation (or recovery)
### Init wallet
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.wallet",
  "content": {
    "msgtype": "io.nunchuk.wallet.init",
    "v": 1,
    "body": {
        "name": "Wallet Name",
        "description": "Wallet description (optional)",
        "m": 2,
        "n": 3,
        "address_type": "NATIVE_SEGWIT", // LEGACY, NESTED_SEGWIT or NATIVE_SEGWIT (default NATIVE_SEGWIT)
        // address_type could be replaced with script_format (P2WSH, P2WSH-P2SH, P2SH)
        "is_escrow": false,
        "member": [], // Optional key whitelisting (if not empty, join event key must belong to this list)
        "chain": "MAIN", // MAIN, TESTNET or REGTEST (default MAIN)
        "bsms": "BSMS 1.0 ..."
    }
  }
}
```

### Join wallet
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io (@liz:nunchuk.io, @niro:nunchuk.io)

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.wallet",
  "content": {
    "msgtype": "io.nunchuk.wallet.join",
    "v": 1,
    "body": {
        "key": "[1cf0bf7e/48'/0'/0'/2']xpub6FL8FhxNNUVnG64YurPd16AfGyvFLhh7S2uSsDqR3Qfcm6o9jtcMYwh6DvmcBF9qozxNQmTCVvWtxLpKTnhVLN3Pgnu2D3pAoXYFgVyd8Yz", // KEY format follow https://github.com/bitcoin/bips/blob/master/bip-0129.mediawiki#signer-1
        // If init_event.content.body.member != [], member must include key
        "type": "HARDWARE", // NFC, HARDWARE, AIRGAP, SOFTWARE or FOREIGN_SOFTWARE
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.wallet",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.wallet.init",
                "v": 1,
                "body": {
                    "name": "Wallet Name",
                    "description": "Wallet description (optional)",
                    "m": 2,
                    "n": 3,
                    "address_type": "NATIVE_SEGWIT",
                    "is_escrow": false,
                    "member": [],
                    "chain": "MAIN",
                    "bsms": "BSMS 1.0 ..."
                }
              }
            }
        }
    }
  }
}
```

### Leave wallet
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io (@liz:nunchuk.io, @niro:nunchuk.io)

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.wallet",
  "content": {
    "msgtype": "io.nunchuk.wallet.leave",
    "v": 1,
    "body": {
        "reason": "",
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.wallet",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.wallet.init",
                "v": 1,
                "body": {
                    "name": "Wallet Name",
                    "description": "Wallet description (optional)",
                    "m": 2,
                    "n": 3,
                    "address_type": "NATIVE_SEGWIT",
                    "is_escrow": false,
                    "member": [],
                    "chain": "MAIN",
                    "bsms": "BSMS 1.0 ..."
                }
              }
            },
            "join_event_id": "$maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM"
        }
    }
  }
}
```

### Wallet ready
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.wallet",
  "content": {
    "msgtype": "io.nunchuk.wallet.ready",
    "v": 1,
    "body": {
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.wallet",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.wallet.init",
                "v": 1,
                "body": {
                    "name": "Wallet Name",
                    "description": "Wallet description (optional)",
                    "m": 2,
                    "n": 3,
                    "address_type": "NATIVE_SEGWIT",
                    "is_escrow": false,
                    "member": [],
                    "chain": "MAIN",
                    "bsms": "BSMS 1.0 ..."
                }
              }
            },
            "join_ids": [
                "$maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM",
                "liz:nunchuk.io_join_event_id",
                "niro:nunchuk.io_join_event_id"
            ]
        }
    }
  }
}
```

### Finalize wallet
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.wallet",
  "content": {
    "msgtype": "io.nunchuk.wallet.create",
    "v": 1,
    "body": {
        "descriptor": "wsh(sortedmulti(1,[59865f44/48'/0'/0'/2']026d15412460ba0d881c21837bb999233896085a9ed4e5445bd637c10e579768ba,[b7044ca6/48'/0'/0'/2']030baf0497ab406ff50cb48b4013abac8a0338758d2fd54cd934927afa57cc2062))#rzx9dffd",
        // (?) maybe support descriptor_template with path_restriction later
        "first_address": "bc1quqy523xu3l8che3s8vja8n33qtg0uyugr9l5z092s3wa50p8t7rqy6zumf",
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.wallet",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.wallet.init",
                "v": 1,
                "body": {
                    "name": "Wallet Name",
                    "description": "Wallet description (optional)",
                    "m": 2,
                    "n": 3,
                    "address_type": "NATIVE_SEGWIT",
                    "is_escrow": false,
                    "member": [],
                    "chain": "MAIN",
                    "bsms": "BSMS 1.0 ..."
                }
              }
            },
            "join_ids": [
                "$maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM",
                "liz:nunchuk.io_join_event_id",
                "niro:nunchuk.io_join_event_id"
            ]
        }
    }
  }
}
```

### Cancel wallet
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.wallet",
  "content": {
    "msgtype": "io.nunchuk.wallet.cancel",
    "v": 1,
    "body": {
        "reason": "",
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.wallet",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.wallet.init",
                "v": 1,
                "body": {
                    "name": "Wallet Name",
                    "description": "Wallet description (optional)",
                    "m": 2,
                    "n": 3,
                    "address_type": "NATIVE_SEGWIT",
                    "is_escrow": false,
                    "member": [],
                    "chain": "MAIN",
                    "bsms": "BSMS 1.0 ..."
                }
              }
            }
        }
    }
  }
}
```

### Delete wallet
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.wallet",
  "content": {
    "msgtype": "io.nunchuk.wallet.delete",
    "v": 1,
    "body": {
        "wallet_id": "5pu4agtf",
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.wallet",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.wallet.init",
                "v": 1,
                "body": {
                    "name": "Wallet Name",
                    "description": "Wallet description (optional)",
                    "m": 2,
                    "n": 3,
                    "address_type": "NATIVE_SEGWIT",
                    "is_escrow": false,
                    "member": [],
                    "chain": "MAIN",
                    "bsms": "BSMS 1.0 ..."
                }
              }
            }
        }
    }
  }
}
```

## Transaction creation
### Init transaction
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.transaction",
  "content": {
    "msgtype": "io.nunchuk.transaction.init",
    "v": 1,
    "body": {
        "wallet_id": "",
        "memo": "Transaction memo (optional)",
        "psbt": "unsigned PSBT encoded in base64",
        "fee_rate": 1,
        "subtract_fee_from_amount": false,
        "chain": "MAIN", // MAIN, TESTNET or REGTEST (default is MAIN)
    }
  }
}
```

### Sign transaction
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io (@liz:nunchuk.io, @niro:nunchuk.io)

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.transaction",
  "content": {
    "msgtype": "io.nunchuk.transaction.sign",
    "v": 1,
    "body": {
        "psbt": "signed PSBT encoded in base64",
        "master_fingerprint": "bc4e023d",
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.transaction",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.transaction.init",
                "v": 1,
                "body": {
                    "wallet_id": "",
                    "memo": "Transaction memo (optional)",
                    "psbt": "unsigned PSBT encoded in base64",
                    "fee_rate": 1,
                    "subtract_fee_from_amount": false,
                    "chain": "MAIN"
                }
              }
            }
        }
    }
  }
}
```

### Transaction ready
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.transaction",
  "content": {
    "msgtype": "io.nunchuk.transaction.ready",
    "v": 1,
    "body": {
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.transaction",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.transaction.init",
                "v": 1,
                "body": {
                    "wallet_id": "",
                    "memo": "Transaction memo (optional)",
                    "psbt": "unsigned PSBT encoded in base64",
                    "fee_rate": 1,
                    "subtract_fee_from_amount": false,
                    "chain": "MAIN"
                }
              }
            },
            "sign_ids": [
                "$maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM",
                "liz:nunchuk.io_sign_event_id",
                "niro:nunchuk.io_sign_event_id"
            ]
        }
    }
  }
}
```

### Broadcast transaction
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.transaction",
  "content": {
    "msgtype": "io.nunchuk.transaction.broadcast",
    "v": 1,
    "body": {
        "tx_id": "",
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.transaction",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.transaction.init",
                "v": 1,
                "body": {
                    "wallet_id": "",
                    "memo": "Transaction memo (optional)",
                    "psbt": "unsigned PSBT encoded in base64",
                    "fee_rate": 1,
                    "subtract_fee_from_amount": false,
                    "chain": "MAIN"
                }
              }
            },
            "sign_ids": [
                "$maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM",
                "liz:nunchuk.io_sign_event_id",
                "niro:nunchuk.io_sign_event_id"
            ]
        }
    }
  }
}
```

### Cancel transaction
- Room ID: !DPZjhLvVqrnlOtrqAy:matrix.org
- Event ID: $maTWq8emJUHXMIWzOpowzgfKp4jBE-i_edNhOwhWKVM
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.transaction",
  "content": {
    "msgtype": "io.nunchuk.transaction.cancel",
    "v": 1,
    "body": {
        "reason": "",
        "io.nunchuk.relates_to": {
            "init_event": {
              "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
              "type": "io.nunchuk.transaction",
              "sender": "@olvaus:nunchuk.io",
              "ts": 1631596260590,
              "event_id": "$5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA",
              "content": {
                "msgtype": "io.nunchuk.transaction.init",
                "v": 1,
                "body": {
                    "wallet_id": "",
                    "memo": "Transaction memo (optional)",
                    "psbt": "unsigned PSBT encoded in base64",
                    "fee_rate": 1,
                    "subtract_fee_from_amount": false,
                    "chain": "MAIN"
                }
              }
            }
        }
    }
  }
}
```

 
