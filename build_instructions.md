# Nunchuk Build Instructions

## Release key and signatures

[Release key](https://keyserver.ubuntu.com/pks/lookup?search=0x8C8ECD3F660CA53CD878792A6E38A462ED2EF525&fingerprint=on&op=index)

Release signatures are attached to each [Nunchuk Desktop release](https://github.com/nunchuk-io/nunchuk-desktop/releases).

## Verifying release signatures

Follow these steps:

1. Download the app file for your operating system and `SHA256SUMS.asc` to the same directory. Open a terminal (or Command Prompt on Windows) and change to that directory.
2. Import the signer's public key:

   ```shell
   gpg --keyserver keyserver.ubuntu.com --recv-keys 0x8C8ECD3F660CA53CD878792A6E38A462ED2EF525
   ```

3. Verify the checksums:

   **Linux and macOS**

   ```shell
   sha256sum --check SHA256SUMS.asc
   ```

   The output should include `OK` for the downloaded app. If you downloaded the app for only one operating system, ignore warnings about files for the other operating systems.

   **Windows**

   ```powershell
   certUtil -hashfile "<downloaded app file>" SHA256
   ```

   The output should match the file's SHA-256 hash in `SHA256SUMS.asc`.

4. Verify the signature:

   ```shell
   gpg --verify SHA256SUMS.asc
   ```

   The output should report a `Good signature` from the signer. Do not continue if the signature cannot be verified.

## Reproducing builds

To reproduce the Linux build and compare it byte for byte with the official release, follow the [Reproducible Builds Guide](https://github.com/nunchuk-io/nunchuk-desktop/blob/main/reproducible-builds/README.md).

## Linux device rules

On Linux, install udev rules so that [HWI](https://github.com/bitcoin-core/HWI) can detect supported hardware devices. Download the rules from HWI and then run the following commands:

```shell
cd hwilib/ && \
  sudo cp udev/*.rules /etc/udev/rules.d/ && \
  sudo udevadm trigger && \
  sudo udevadm control --reload-rules && \
  sudo groupadd plugdev && \
  sudo usermod -aG plugdev "$(whoami)"
```

See [HWI's udev documentation](https://github.com/bitcoin-core/HWI/tree/master/hwilib/udev) for more information.
