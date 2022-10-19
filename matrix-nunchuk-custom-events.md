# Matrix Nunchuk custom events

## Create Wallet
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
        // (?) maybe replace address_type by script_format (P2WSH, P2WSH-P2SH, P2SH)
        "is_escrow": false,
        "member": [], // Key whitelist (if not empty, join event key must belong to member list)
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
        "type": "HARDWARE", // HARDWARE, SOFTWARE, AIRGAP or FOREIGN_SOFTWARE
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

## Create Transaction
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
        "chain": "MAIN", // MAIN, TESTNET or REGTEST (default MAIN)
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

## Synchronize Multiple Device
### Backup
- Room ID: <special sync room>
- Event ID: $5NCjFf5HI6yVlgBhBHo2_qAqTZWs6MXKJh3QUPc9AsA
- Sender: @olvaus:nunchuk.io

```json
{
  "room_id": "!DPZjhLvVqrnlOtrqAy:matrix.org",
  "type": "io.nunchuk.sync",
  "content": {
    "msgtype": "io.nunchuk.sync.file",
    "v": 1,
    "file": {
        "v": "v2",
        "key": {
            "alg": "A256CTR",
            "ext": true,
            "k": "key encoded in base64",
            "key_ops": ["encrypt", "decrypt"],
            "kty": "oct"
        },
        "iv": "iv encoded in base64",
        "url": "",
        "mimetype": "application/octet-stream",
        "hashes": {
            "sha256": "sha256 encoded in base64"
        }
    }
  }
}
```
 
