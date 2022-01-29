# Nunchuk Build Instructions

## Release key and signatures

[Release key](https://keyserver.ubuntu.com/pks/lookup?search=0x8C8ECD3F660CA53CD878792A6E38A462ED2EF525&fingerprint=on&op=index)

[Release signatures](https://nunchuk-downloads.s3.ap-southeast-1.amazonaws.com/v1.9.4/SHA256SUMS.asc)

## Verifying builds

Follow these steps:

1. Download the app for your OS (.zip) and the signature file (.asc) into the same directory, then open the Command Line and `cd` into the directory.
2. Import the public key of our signer. The signing key can be found above. 

    `gpg --keyserver keyserver.ubuntu.com --recv-keys 0x8C8ECD3F660CA53CD878792A6E38A462ED2EF525`

3. Verify the checksum

    `sha256sum --check SHA256SUMS.asc`

The output should say "OK" if the checksum is valid for the given file (if you only download the app for one OS, ignore the warnings for the other OSes).


4. Verify the signature

    `gpg --verify SHA256SUMS.asc`

The output should say "Good signature" from our signer.

## Linux

On Linux, you will need to install udev rules for the devices to be reachable by [HWI](https://github.com/bitcoin-core/HWI). Get the rules from HWI then run the following command: 

```Shell
cd hwilib/; \
  sudo cp udev/*.rules /etc/udev/rules.d/ && \
  sudo udevadm trigger && \
  sudo udevadm control --reload-rules  && \
  sudo groupadd plugdev && \
  sudo usermod -aG plugdev `whoami`
```

Visit [here](https://github.com/bitcoin-core/HWI/tree/master/hwilib/udev) for more information.
