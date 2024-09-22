# ACPI fix for KC57

This is a set of ACPI fixes for INTEL NUC X15(LAPKC5\*, LAPKC7\*).

## Backgroud
Original DSDT is written against Windows OS, so there are some conflicts on Linux, like

```
[    0.471306] pnp 00:03: disabling [mem 0xc0000000-0xcfffffff] because it overlaps 0000:00:02.0 BAR 9 [mem 0x00000000-0xdfffffff 64bit pref]
...
[    3.042108] resource: resource sanity check: requesting [mem 0x00000000fedc0000-0x00000000fedcdfff], which spans more than pnp 00:03 [mem 0xfedc0000-0xfedc7fff]
[    3.042112] caller __uncore_imc_init_box+0xd9/0x120 [intel_uncore] mapping multiple BARs
```
and I found
```
[    0.480131] pci 0000:00:02.0: VF BAR 2 [mem 0x4020000000-0x40ffffffff 64bit pref]: assigned
```

newer intel integrated graphics support [SR-IOV](https://open-iov.org/index.php/GPU_Support), and they need reserve space(\[mem 0x00000000-0xdfffffff\]) for 7 VFs(virtual function), which causes memory overlap, but they actually use a block of 64bit memory space(\[mem 0x4020000000-0x40ffffffff\]), `pnp 00:03` has no impact on it. So we can cancel the [disabling mechanism](https://elixir.bootlin.com/linux/v6.11/source/drivers/pnp/quirks.c#L285), or chose another piece of memory block for `pnp 00:03`.

Another issue is that one of `pnp 00:03`'s resources(\[mem 0xfedc0000-0xfedc7fff\]) spans more than requested, there is a potential overflow, we can solve it by requesting the correct size for it.

It is said that they are unharmful, however, I expirenced machine freezing frequently when I shut down from my DE(KDE Plasma, Arch Linux). Now I fixed it, nowadays, I have not met it again.

## How to apply
They are written against BIOS version 40, if yours is different from mine, you should do some tweaks.

```bash
# access your DSDT table
cat /sys/firmware/acpi/tables/DSDT > dsdt.dat
# disassemble it
iasl -d dsdt.dat
# apply patches
git apply ./000*
# assemble it
iasl -ve dsdt.dsl
```

After this, you should override DSDT table at boot time. There are many ways you can find.
