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
