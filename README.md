# Custos OS — operating system

*custōs* · Latin for **guardian, keeper**, the root of the word *custody*.

The operating-system half of [Custos OS](https://custos-os.com): a fork of
[SeedSigner OS](https://github.com/SeedSigner/seedsigner-os) that adds the
smartcard stack needed to read and write SeedKeeper cards on a Raspberry Pi Zero,
while keeping SeedSigner OS's minimal, network-free, RAM-only design.

The app is built from
[custos-seedsigner](https://github.com/xmrstreetstore/custos-seedsigner), and
release images, build scripts and verifiers are in
[custos-os](https://github.com/xmrstreetstore/custos-os).

> **Status: v0.1, beta.** Only the **pi0** target (Raspberry Pi Zero v1.3) is built
> and tested. The other boards received the SEC1210 configuration but have not been
> built or run.

## What it adds to SeedSigner OS

**Contact reader: SEC1210 smartcard HAT.** The SEC1210 is a *serial* CCID reader on
`/dev/ttyAMA0`, so:

- CCID is built with serial support and pcsc-lite reads it through
  `etc/reader.conf.d/sec1210` (`libccidtwin.so`).
- The PL011 UART driver is built in, `dtoverlay=disable-bt` frees the UART, and the
  serial console is kept off it.
- pcscd is patched so it carries on with serial readers when there is no USB, and
  `etc/init.d/S01pcscd` starts it at boot.
- `CONFIG_UNIX` is enabled on pi0. pcsc-lite talks over an AF_UNIX socket, and the
  stock pi0 kernel has none, so without it pcscd dies on start. This is local IPC
  only, with no network interface or driver.

**Contactless reader: PN532 over I2C (pi0).** libnfc with the `pn532_i2c` driver, and
`ifdnfc` as a second pcscd reader named `IFD-NFC` (`etc/nfc/libnfc.conf`,
`etc/reader.conf.d/libifdnfc`). I2C bus 1 is enabled, since the UART belongs to the
SEC1210. Either reader can be switched off in the app.

**Python packages:** `pysatochip` and its dependencies, `pyscard` and `pygp`. pygp's
cryptography dependency is replaced by a small backend on pycryptodomex (already
present for pysatochip), so the image does not need Rust-built `cryptography`.

**pi4-dev** gains the Pi 4's own ethernet driver, so a development board can be
reached over SSH.

## What it keeps

No Wi-Fi, no Bluetooth: the release image builds no radio drivers and ships no
Broadcom wireless firmware, and the Pi Zero v1.3 has no wireless chip. NFC is the
one RF link, short-range and card-only. The system runs from RAM; nothing is stored.

## Building

Builds use the stock SeedSigner OS Docker flow ([docs/building.md](docs/building.md)),
pointed at the Custos app. Inside the build container, `opt/build.sh` takes:

```bash
./build.sh --pi0 --app-repo=https://github.com/xmrstreetstore/custos-seedsigner.git --app-branch=seedkeeper-support
```

`--app-repo` and `--app-branch` only parse in the `--flag=value` form. The
[custos-os](https://github.com/xmrstreetstore/custos-os) repo has the scripts used
for the released image, which build inside a Docker volume, and the verifiers that
check the result. The stock README is kept as
[README.seedsigner-os.md](README.seedsigner-os.md).

## Licence and credits

MIT, like SeedSigner OS — see [LICENSE.md](LICENSE.md). Built on SeedSigner OS by
the SeedSigner contributors. The smartcard packaging follows
[3rdIteration's SeedSigner OS fork](https://github.com/3rdIteration/seedsigner-os).
Packages built into the image keep their own licences.

Custos OS is an independent project, not affiliated with SeedSigner or Satochip.
