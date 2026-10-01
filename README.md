# OS/2 Warp PowerPC Edition on QEMU

Patches against QEMU 10.1.0 that add `carolina`, an IBM Power Series 850, and
the fixes needed to run the original IBM firmware and OS/2 Warp, PowerPC
Edition (December 1995) on it: installation from CD, boot from disk, and the
Workplace Shell with the onboard S3 Vision864.

## Patches

| Patch | What it does |
|---|---|
| 0001 | 60x little-endian mode: address munging, alignment interrupts with the 60x DSISR/DAR, ILE kept on exception entry, `rfi` loads only MSR[16-31] |
| 0002 | Flush the TLB on `mtsr`/`mtsrin`; the microkernel copies through SR0 right after changing it, without `isync` |
| 0003 | IDE: an interrupt raised while nIEN is set stays pending until nIEN is cleared; IDENTIFY does not advertise DMA on a bus without it; ATAPI READ CD (0xD4) |
| 0004 | FDC: seek and recalibrate complete 5 ms later instead of instantly |
| 0005 | IBM FireCoral ISA bridge (1014:000a), PC87312 reconfiguration, PReP system I/O little-endian bit |
| 0006 | S3 Vision864 with S3 SDAC (5333:88c0): extended CRTC, linear frame buffer, 8514-style drawing engine, hardware cursor, 8/16/32 bpp. Weitek P9100 (by Ørjan Malde) |
| 0007 | The `carolina` machine: IBM 660 bridge, memory banks, NVRAM with factory layout, IDE interrupt latch, NMI into the microkernel debugger |

## Build

```
tar xf qemu-10.1.0.tar.xz
cd qemu-10.1.0
for p in ../patches/*.patch; do patch -p1 < $p; done
mkdir build && cd build
../configure --target-list=ppc-softmmu
ninja qemu-system-ppc
```

## Run

The firmware is `P93H1904.IMG` (Power Series 850/830, version 1.10), inside
`p8x0_110.exe` in IBM's `850_830_firmware.zip`. Put it in a directory passed
with `-L`.

Install (CD 1 in the drive, empty disk; the firmware boots the CD on its own):

```
qemu-img create -f qcow2 os2ppc.qcow2 700M
qemu-system-ppc -M carolina -L fw -nic none \
  -drive file=os2ppc.qcow2,if=ide,index=0,media=disk \
  -drive file=os2-ppc-cd1-os.iso,if=ide,index=2,media=cdrom,readonly=on \
  -global carolina-nvram.file=nvram.bin
```

Afterwards boot with the CD drive empty
(`-drive if=ide,index=2,media=cdrom`). The desktop appears after three to four
minutes.

`-machine video=p9100` replaces the S3 with a Weitek P9100 in slot 13. OS/2
reaches the desktop with it, but its driver draws no mouse pointer.

`S3_DEBUG=1` in the environment logs every drawing command (`-d` log).

## Known limitations

- OS/2 Warp PowerPC Edition has no networking: IBM's own brochure says so.
  Hence `-nic none`.
- No sound.
- OS/2 does not keep the mouse tracking speed across boots; it starts at the
  fastest setting.
- Tested modes: 640x480 at 8, 16 and 32 bpp, 800x600x8, 1024x768x16,
  1280x1024x8.

## Where to look

- Media: [archive.org/details/os2-powerpc](https://archive.org/details/os2-powerpc)
  (CD 1 is the system, CD 2 an application sampler).
- Firmware: [ibmfiles.com, 850_830_firmware.zip](http://www.ibmfiles.com/ibmfiles/powerpc/850_830_firmware.zip).
- The P9100 model comes from Ørjan Malde's QEMU for Windows NT on PowerPC,
  [hiirotsuki/qemu-windows-nt-ppc](https://github.com/hiirotsuki/qemu-windows-nt-ppc).
- The S3 SDAC and hardware cursor behaviour was checked against
  [86Box](https://github.com/86Box/86Box).

## License

GPL-2.0, as QEMU. See [LICENSE](LICENSE). `p9100.c` is GPL-2.0-or-later.

Written with Claude Opus 5.5 (model claude-opus-5-5, Anthropic) through Claude Code.
