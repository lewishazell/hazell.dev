+++
title = "Unlocking LUKS volumes with your OnlyKey"
date = "2024-09-14"

[taxonomies]
tags=["onlykey", "luks", "setup", "fido2", "diceware", "systemd"]

[extra]
repo_view = true
+++

A while ago, I purchased an OnlyKey Duo. I intended to use it for GPG, SSH and MFA. Alas, I still have very little use for GPG. However, I'm pretty tired of typing in a sufficiently long [diceware](https://www.eff.org/dice) passphrase when I start my laptop.

The [OnlyKey documentation](https://docs.onlykey.io/full-disk-encryption.html) is pretty limited on this topic and links to a repository which is still in alpha. Can we do better?

As it turns out, you can use `systemd-cryptenroll` with the OnlyKey's FIDO2 capabilities for a well supported solution.

# Pre-setup
I'm assuming you already have a LUKS2 volume set up with a passphrase. If you don't, I recommend you do as a passphrase will allow you to access your machine if you lose your OnlyKey.

If you want to remove passphrases, I recommend taking a recovery key in its place and keeping it somewhere safe:
```
$ sudo systemd-cryptenroll --recovery-key /dev/sda3
```
(Replace _/dev/sda3_ with the underlying block device of your encrypted volume)

## systemd version
For this setup to work, you need _systemd_ version 248 or above, you can check this with
```
$ systemctl --version
```

For example, I am running version 255
> systemd 255 (255.10-3.fc40)

## Disable GRUB menu timeout
A problem you will face is that you need to unlock your OnlyKey before user presence is requested. It's only requested once and doesn't leave enough time to input your PIN.

I found that the GRUB boot menu is the perfect opportunity to unlock my OnlyKey and proceed when I'm ready. User presence isn't requested until Linux boots.

To allow yourself time, disable the GRUB menu timeout by setting `GRUB_TIMEOUT` to `-1` in _/etc/default/grub_:
```
$ sudoedit /etc/default/grub
[sudo] password for user:

# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
...
GRUB_TIMEOUT=-1
...
```

After modifying _/etc/default/grub_, you need to update _/boot/grub/grub.cfg_ to reflect your changes. Do so by running `update-grub`:
```
$ sudo update-grub
```

# Setup
With that out of the way, let's enroll the OnlyKey with your LUKS2 volume, configure crypttab to enable FIDO2 unlock and regenerate initramfs.

## Enroll your token
To enroll your OnlyKey as an additional way to unlock your volume, run the command
```
$ sudo systemd-cryptenroll --fido2-device=auto --fido2-with-client-pin=false --fido2-with-user-presence=true /dev/sda3
```

(Replace _/dev/sda3_ with the underlying block device of your encrypted volume)

This embeds the information for FIDO2 unlock with your OnlyKey in the LUKS2 header of your volume.

I have set `--fido2-with-client-pin=false` because I rely on the OnlyKey hardware PIN input. If you have also set a client PIN, you will want to set this to `true`.

## Allow FIDO2 unlocking via crypttab
For FIDO2 unlock to work, it needs to be enabled in _/etc/crypttab_ by appending the `fido2-device=auto` option on the line configuring your volume:
```
$ sudoedit /etc/crypttab
[sudo] password for user:

...
myvolume /dev/sda3 - fido2-device=auto
...
```
(_/dev/sda3_ will appear as your block device, it may also be expressed as a UUID)

## Regenerate initramfs
Before rebooting your machine, regenerate your initramfs by running:
```
$ sudo dracut --regenerate-all --force
```

Now it's all ready to rock. Reboot your machine, enter your OnlyKey PIN, select your distro and your OnlyKey should flash blue to unlock the machine!

# Teardown
Just for completeness, you can remove your OnlyKey as a key for your LUKS volume.

Given this is your first token, it's probably going to have an ID of `0`. You can remove it from the LUKS volume by running:
```
$ sudo cryptsetup token remove --token-id 0 /dev/sda3
``` 
and then remove the corresponding key slot by running:
```
$ sudo systemd-cryptenroll --wipe-slot=1 /dev/sda3
```
(Replace _/dev/sda3_ with the underlying block device of your encrypted volume)