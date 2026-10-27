# oubliette-patches

unofficial Portage user patches for Gentoo Linux, this repo supplements [oubliette-overlay](https://github.com/nabbi/oubliette-overlay)

## install

```shell
sudo git clone https://github.com/nabbi/oubliette-patches.git /etc/portage/patches
```

## notes

### sys-fs/cryptsetup: duress keyslot

`sys-fs/cryptsetup/cryptsetup-2.8.8-duress-keyslot.patch` adds a LUKS2
duress keyslot: if a passphrase fails to unlock a device but opens an
**unbound keyslot 31**, every keyslot's key material is overwritten and an
ordinary wrong-passphrase error is shown. It does nothing unless slot 31 is
an unbound keyslot. Arm it, and check the duress passphrase without wiping:

```sh
cryptsetup luksAddKey --unbound --key-size 256 --key-slot 31 <device>
cryptsetup open --test-passphrase --key-slot 31 <device>
```

Any `open` without `--key-slot` that reaches slot 31 wipes. Keep a header
backup (`cryptsetup luksHeaderBackup`); it is the only way back.

The directory is unversioned on purpose: a cryptsetup bump the patch no longer
applies to fails to build rather than silently dropping the feature. With
dracut, regenerate the initramfs after updating cryptsetup.

### sys-auth/pam_u2f: manual mode without CBOR parsing of the paste

`sys-auth/pam_u2f/pam_u2f-1.4.1_rc1-manual-raw-authdata.patch` makes
`manual` mode unwrap the pasted authenticator data's CBOR byte-string
header itself and hand the raw bytes to libfido2, so the remote user's
bytes never reach libcbor's `cbor_load()` (PJK/libcbor#418 class). Upstream
PR: Yubico/pam-u2f#396; this copy is the same commit rebased onto the fork
tag the overlay ebuild builds (nabbi/pam-u2f `ship/manual-raw-authdata-1.4.1_rc1`,
04c29ab). The fleet is already covered by the fork's `RLIMIT_AS` guard and
by `dev-libs/libfido2[cbor-hardening]`; this is the third, smallest layer.
Drop it when the overlay bumps to a fork tag that includes it.

### dev-libs/libcbor: size definite containers by what the input can hold

`dev-libs/libcbor/libcbor-claim-size.patch` stops `cbor_load()` from
preallocating more array or map elements than the remaining input could
hold. Upstream PR: PJK/libcbor#456. Applies to v0.14.0 and to master as of
3611838, which `libcbor-9999` builds. Both directories are unversioned so
a bump the patch no longer applies to fails loudly. Note for the dev box:
with this applied, the libcbor allocation is no longer reproducible there
through `libfido2[fuzz -cbor-hardening]`; remove the file locally if that
reproduction is needed.
