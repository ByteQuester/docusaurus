---
id: sops-with-age
slug: /integrations/sops-with-age
sidebar_position: 11
---

# SOPS with age

:::info This document explains how to use SOPS with age to encrypt and decrypt secrets in this repository. :::

## 1. Generate an age key

If you don't have an age key, you can generate one with the `age-keygen` command.

```bash title="Generate age key"
age-keygen -o key.txt
```

This will create a file `key.txt` with your public and private keys.

:::danger Secure Your Private Key Your `key.txt` file contains your private key. Keep it safe and do not commit it to the repository. :::

## 2. Configure SOPS

The `.sops.yaml` file in the root of the repository is configured to use your age public key. You will need to replace the placeholder public key with your own.

## 3. Encrypting a File

To encrypt a file, use the `sops` command. This will encrypt the `data` and `stringData` fields in the file.

```bash title="Encrypt a file in-place"
sops --encrypt --in-place my-secrets.yaml
```

## 4. Decrypting a File

To decrypt a file, use the `sops` command with your private key. Your private key will be used to decrypt the file.

```bash title="Decrypt a file in-place"
sops --decrypt --in-place my-secrets.yaml
```
