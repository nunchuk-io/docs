# Nunchuk Build Instructions

## Signed Builds

[Release key](https://keyserver.ubuntu.com/pks/lookup?search=0x8C8ECD3F660CA53CD878792A6E38A462ED2EF525&fingerprint=on&op=index)

[Release signatures](https://nunchuk.io/downloads/v1.0.1/SHA256SUMS.asc)

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
