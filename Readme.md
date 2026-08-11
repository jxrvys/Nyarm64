# Nyarm64 (ARM64 Nyarch Linux) — Generic Rootfs

An ARM64 build of Nyarch Linux, built and
tested primarily under QEMU (`virt` machine, TCG emulation) on an x86_64 host.

This is **not** a bootable disk image. It's a root filesystem tarball bcs i cant spend the time to make a bazillion different versions.

## What's included

- Arch Linux ARM generic aarch64 base
- GNOME desktop (`gnome` + `gnome-extra` package groups)
- NetworkManager, systemd-timesyncd (enabled)
- Nyarch Assistant
- Catgirl/Waifu Downloader

## What's *not* included / pain you are forced to endure / limitations (go buy a fucking x86_64 device pls so you dont have to use this crap :sob:)

- **No kernel or bootloader.** You must supply one appropriate for your
  device
- **Live2D avatar animation in Nyarch Assistant does not currently work.**
  The chat/text functionality works correctly; the animated avatar throws
  JS errors in its WebKitGTK view (`get_expressions_json` / motion-related
  errors) that haven't been root-caused yet.
- **Optional Nyarch Assistant features** (web search via `ddgs`, RAG/document
  search via `llama-index`/`faiss`) fail to auto-install at runtime due to
  Python's PEP 668 "externally managed environment" protection on Arch. Core
  chat works regardless.
- This has **not** been tested on real ARM64 hardware** — only under QEMU
  software emulation. Expect device-specific surprises that this build hasn't encountered.

## enhancing weebness

These are general steps — the specifics depend heavily on your hardware.

1. **Get a working kernel + bootloader for your device first**, independent
   of this tarball. E.g.:
   - Raspberry Pi: `linux-rpi` package + the Pi's own firmware boot chain
     (`config.txt`, `bootcode.bin`, etc.)
   - Generic UEFI ARM64 board: GRUB arm64-efi
   - QEMU/VM: boot directly via `-kernel`/`-initrd`
2. **Partition your target storage** with at least an ESP/boot partition (if
   your boot chain needs one) and a root partition (ext4 recommended).
3. **Extract this tarball as the root partition's contents**, preserving
   permissions and ownership:
   ```bash
   sudo tar --numeric-owner -xzf nyarch-arm64-rootfs-v2.tar.gz -C /path/to/mounted/root
   ```
4. **Install your kernel/bootloader into that same rootfs** (via chroot, or
   however your device's normal Arch Linux ARM install process works).
5. **Boot it.** On first boot:
   - Create a user: `useradd -m -G wheel -s /bin/bash yourname && passwd yourname`
   - Enable sudo: uncomment the `%wheel` line in `/etc/sudoers`
   - Confirm networking and `systemd-timesyncd` are enabled and synced
     (`timedatectl status`)
   - Log in (duh)

## Stupid shit

**The installed kernel package version must match the actual kernel you're
booting.** If you run `pacman -Syu` and it updates `linux-aarch64` (or
whichever kernel package you're using) *after* you've already extracted
`/boot/Image` for use elsewhere, the module directory under
`/usr/lib/modules/<version>/` gets replaced with the new version's modules —
but the kernel you're actually booting is still the old one because ts fucking sucks. This manifests
as `modprobe: FATAL: Module <x> not found`, a GPU that never initializes, and GDM silently failing to start a graphical
session with no clear error. Always re-copy `/boot/Image` and
`/boot/initramfs-*.img` *after* any package update, before rebooting.

## annoying crap that i probably wont do

- Root-cause the Live2D avatar WebKit errors
- Fix optional-dependency auto-install under PEP 668 (likely needs the app's
  pip-install logic patched to use `--break-system-packages` or a venv)

## the providers of my dumb obsessions

Built on [Arch Linux ARM](https://archlinuxarm.org/) and
[NyarchLinux](https://github.com/NyarchLinux).
