---
title: "RSA Shenanigans"
date: 2023-07-26T11:25:40+02:00
draft: false
hidden: true
author: Lea
author_url: https://lea.pet/@lea
---

If you found this, cool! It's not meant to be a guide - I just created this page so I don't have to painstakingly figure all of this out again.

### Create PKCS12 file from certificate and private key

```bash
openssl pkcs12 -export -in pki/issued/mycert.crt -inkey pki/private/mycert.key -name 'mycert' -out mycert.p12
```

## Client certificates on YubiKey
> See https://johanfagerstroem.github.io/2021/https-client-certificates-with-yubikeys/

### Setting up smart card service

```bash
pacman -Syu ccid opensc
paru -Syu yubikey-manager
systemctl enable --now pcscd
```

### Change the default management key

You should do this before doing cryptography shenanigans with it

```bash
ykman piv access change-management-key --generate --protect
```

### Generate keys

```bash
# Generate private key and output public key
ykman piv keys generate 9a yubikey.pub

# Create a signing request from the previously generated key
ykman piv certificates request -s yubikey 9a yubikey.pub yubikey.csr
```

### Sign the request

```bash
easyrsa import-req ./yubikey.csr yubikey
easyrsa sign-req client yubikey

# Cert is now at pki/issued/yubikey.crt.
# Let's import the public key into the device:
ykman piv certificates import 9a pki/issued/yubikey.crt
```

### Use the Yubikey in Firefox

Go to `Settings` > `Privacy & Security` > `Certificates` > `Security Devices`

{{< image "rsa-shenanigans/ff-settings-1.png" >}}

Click "Load" and enter the path to `libykcs11.so`. On my system it's at `/usr/lib/libykcs11.so`, try `find / -name libykcs11.so 2>/dev/null` if you can't find it.

> *ℹ️ libykcs11 is provided by `yubico-piv-tool`*

{{< image "rsa-shenanigans/ff-settings-2.png" >}}

### If Firefox stops prompting for a certificate...
Check that your key is plugged in. If not, there will be no error - The page will just refuse to load until you plug it in.
If it still doesn't work, go to `View Certificates` > `Authentication Decisions` and delete the relevant entry.

{{< image "rsa-shenanigans/ff-settings-3.png" >}}
