---
title: crypto_RSA_OAEP
---

## Description

This plugin allows to encrypt/decrypt strings with asymmetric RSA keys, using PKCS#1 OAEP, an asymmetric cipher based on RSA and the OAEP padding.

## Configuration

You can set up as many keys as you want in plugin settings. A key can be used to either encrypt or decrypt, but not both. The plugin will determine if it's an encryption or a decryption operation upon the key type: decryption for private keys, and encryption for public keys.

```yaml
hermes:
  plugins:
    attributes:
      crypto_RSA_OAEP:
        settings:
          keys:
            # Key name, you can set whatever you want
            encrypt_to_messagebus:
              # Hash type, when decrypting, you must obviously use the same value
              # that was used for encrypting
              hash: SHA3_512
              # Public RSA key used to decrypt
              # WARNING - THIS KEY IS WEAK AND PUBLIC, NEVER USE IT
              rsa_key: |-
                  -----BEGIN PUBLIC KEY-----
                  MCgCIQCy2W1bAPOa1JIeLuV8qq1Qg7h0jxpf8QCik11H9xZcfwIDAQAB
                  -----END PUBLIC KEY-----

            # Another key
            decrypt_from_messagebus:
              hash: SHA3_512
              # Private RSA key used to decrypt
              # WARNING - THIS KEY IS WEAK AND PUBLIC, NEVER USE IT
              rsa_key: |-
                  -----BEGIN RSA PRIVATE KEY-----
                  MIGrAgEAAiEAstltWwDzmtSSHi7lfKqtUIO4dI8aX/EAopNdR/cWXH8CAwEAAQIh
                  AKfflFjGNOJQwvJX3Io+/juxO+HFd7SRC++zBD9paZqZAhEA5OtjZQUapRrV/aC5
                  NXFsswIRAMgBtgpz+t0FxyEXdzlcTwUCEHU6WZ8M2xU7xePpH49Ps2MCEQC+78s+
                  /WvfNtXcRI+gJfyVAhAjcIWzHC5q4wzgL7psbPGy
                  -----END RSA PRIVATE KEY-----
```

Valid values for `hash` are:

- SHA224
- SHA256
- SHA384
- SHA512
- SHA3_224
- SHA3_256
- SHA3_384
- SHA3_512

## Usage

```python
crypto_RSA_OAEP(value: bytes | str, keyname: str) → str
```

Once everything is set up, you can encrypt data with `encrypt_to_messagebus` key like this  in a Jinja filter:

```yaml
password_encrypted: "{{ PASSWORD_CLEAR | crypto_RSA_OAEP('encrypt_to_messagebus') }}"
password_decrypted: "{{ PASSWORD_ENCRYPTED | crypto_RSA_OAEP('decrypt_from_messagebus') }}"
```

You can even decrypt and immediately re-encrypt data with another key like this:

```yaml
password_reencrypted: "{{ PASSWORD_ENCRYPTED | crypto_RSA_OAEP('decrypt_from_datasource') | crypto_RSA_OAEP('encrypt_to_messagebus') }}"
```
