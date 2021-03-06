# wolfBoot Signing

Instructions for setting up Python, wolfCrypt-py module and wolfBoot for firmware signing. 

Note: There is a pure C signing tool available as well. See [C Signing Tool](#c-signing-tool) below.

## Install Python3

1. Download latest Python 3.x and run installer: https://www.python.org/downloads
2. Check the box that says Add Python 3.x to PATH

## Install wolfCrypt

```sh
git clone https://github.com/wolfSSL/wolfssl.git
cd wolfssl
./configure --enable-keygen --enable-rsa --enable-ecc --enable-ed25519 --enable-des3 CFLAGS="-DWOLFSSL_PUBLIC_MP"
make
sudo make install
```

## Install wolfcrypt-py

```sh
git clone https://github.com/wolfSSL/wolfcrypt-py.git
cd wolfcrypt-py
sudo USE_LOCAL_WOLFSSL=/usr/local pip3 install .
```

## Install wolfBoot

```sh
git clone https://github.com/wolfSSL/wolfBoot.git
cd wolfBoot
git submodule update --init
# Setup configuration (or copy template from ./config/examples)
make config
# Build the wolfBoot binary and sign an example test application
make
```

## Signing Firmware

1. Load the private key to use for signing into “./rsa4096.der”
2. `python3 ./tools/keytools/sign.py --rsa4096 --sha3 test-app/image.bin rsa4096.der 1`

Note: The last argument is the “version” number.

## Signing Firmware with External Private Key (HSM)

I've tested this with separate signature and the correct public key, the two files are identical either if I do one step signing:

```sh
# Create file with Public Key
openssl rsa -inform DER -outform DER -in rsa4096.der -out rsa4096_pub.der -pubout

# Generate Hash to Sign
python3 ./tools/keytools/sign.py --rsa4096 --sha-only --sha3 test-app/image.bin rsa4096_pub.der 1

# Example for signing
openssl rsautl -sign -keyform der -inkey rsa4096.der -in test-app/image_v1_digest.bin > test-app/image_v1.sig

# Generate final signed binary
python3 ./tools/keytools/sign.py --rsa4096 --sha3 --manual-sign test-app/image.bin rsa4096_pub.der 1 test-app/image_v1.sig
```

## C Signing Tool

A standalone C version of the signing tool is available here: `./tools/keytools/sign.c`. Build using `make keytools`

```sh
./tools/keytools/sign --rsa4096 --sha3 test-app/image.bin rsa4096.der 1
```

### Windows Visual Studio

Use the `wolfBootSignTool.vcxproj` Visual Studio project to build the `sign.exe` tool for use on Windows.


## Command Line Usage

```
./tools/keytools/sign [--ed25519 | --ecc256 | --rsa2048 | --rsa4096 ] [--sha256 | --sha3] [--wolfboot-update] image key.der fw_version
  - or -        ./tools/keytools/sign [--sha256 | --sha3] [--sha-only] [--wolfboot-update] image pub_key.der fw_version
  - or -        ./tools/keytools/sign [--ed25519 | --ecc256 | --rsa2048 | --rsa4096 ] [--sha256 | --sha3] [--manual-sign] image pub_key.der fw_version signature.sig
```
