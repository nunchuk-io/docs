# Nunchuk Build Instructions

## Release key and signatures

[Release key](https://keyserver.ubuntu.com/pks/lookup?search=0x8C8ECD3F660CA53CD878792A6E38A462ED2EF525&fingerprint=on&op=index)

Release signatures are attached to each [Nunchuk Desktop release](https://github.com/nunchuk-io/nunchuk-desktop/releases).

## Verifying release signatures

Follow these steps:

1. Download the app file for your operating system, `SHA256SUMS`, and `SHA256SUMS.asc` from the same release to the same directory. Open a terminal (or Command Prompt on Windows) and change to that directory.
2. Import the signer's public key:

   ```shell
   gpg --keyserver keyserver.ubuntu.com --recv-keys 0x8C8ECD3F660CA53CD878792A6E38A462ED2EF525
   ```

   Confirm that GPG reports the key was imported, updated, or unchanged.

3. Verify the checksums:

   **Linux**

   ```shell
   sha256sum --check --ignore-missing SHA256SUMS
   ```

   **macOS**

   ```shell
   shasum -a 256 --ignore-missing --check SHA256SUMS
   ```

   Confirm that the output shows `OK` after the name of the app file you downloaded.

   **Windows**

   Replace `<downloaded app file>` with the name of the file you downloaded:

   ```powershell
   certUtil -hashfile "<downloaded app file>" SHA256
   ```

   Display the expected checksums:

   ```powershell
   type SHA256SUMS
   ```

   Confirm that the entire SHA-256 hash from `certUtil` matches the entry for your downloaded file in `SHA256SUMS`.

4. Verify the signature:

   ```shell
   gpg --verify SHA256SUMS.asc SHA256SUMS
   ```

   Confirm that GPG reports a `Good signature` and that the primary key fingerprint matches:

   ```text
   8C8E CD3F 660C A53C D878  792A 6E38 A462 ED2E F525
   ```

   GPG may warn that the key is not certified with a trusted signature. This means GPG has not established trust in the key's identity; it does not by itself mean the signature is invalid. Confirm the fingerprint against a trusted Nunchuk source.

   Do not install or run the app unless the checksum, signature, and fingerprint checks all pass.

## Reproducing builds

To reproduce a build and compare it byte for byte with the official release, follow the [Reproducible Builds Guide](https://github.com/nunchuk-io/nunchuk-desktop/blob/main/reproducible-builds/README.md) for supported platforms and instructions.

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
