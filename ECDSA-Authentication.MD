- [Preface](#preface)
- [Authentication using ECDSA](#authentication-using-ecdsa)
    - [1. Create an identity](#1-create-an-identity)
    - [2. Create a nonce](#2-create-a-nonce)
    - [3. Sign nonce with private key](#3-sign-nonce-with-private-key)
    - [4. Authenticate using signed message](#4-authentication-using-signed-message)
        - [4.1 Sign-up](#41-sign-up)
        - [4.2 Sign-in](#42-sign-in)

# Preface

# Authentication using ECDSA

![Sequence Diagram](ECDSA_authentication.png)

## 1. Create an identity

    # We use bitcoin-cli here, but feel free to use any other tools
    > bitcoin-cli getnewaddress
    
    < 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzN

## 2. Create a nonce

    # identity = hash(public_key). In case of bitcoin-cli, we use the address, which was already hashed.
    # username is an alias to identity
    > GET /account/v1/nonce?identity=${identity}&username=${username}

    < {"nonce":"thisisanonce"}

## 3. Sign nonce with private key

    # We use bitcoin-cli here, but feel free to use any other tools
    # message should be the combination of username and nonce. eg: ${username}${nonce}
    > bitcoin-cli signmessage "${identity}" "${message}"

## 4. Authentication using signed message

### 4.1 Sign-up

    > POST /account/v1/sign-up
    {
        "identity": "${identity}",
        "username": "${username}",
        "nonce": "${nonce}",
        "signature": "${signature}"
    }

    # Response really depends on the server implementation. But usually, it's a access_token, token type and expiration time.
    < {"access_token": "thisisanaccess_token", "token_type": "Bearer", "expires_in": 3600}

### 4.2 Sign-in

1. Server check where (identity, username) is in the database
2. Verify signature

   ```
   > POST /account/v1/sign-up
   {
       "identity": "${identity}",
       "username": "${username}",
       "nonce": "${nonce}",
       "signature": "${signature}"
   }

   # Response really depends on the server implementation. But usually, it's a access_token, token type and expiration time.
   < {"access_token": "thisisanaccess_token", "token_type": "Bearer", "expires_in": 3600}
   ```   
