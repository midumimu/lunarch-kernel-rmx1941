# Lunarch Kernel — RMX1941

Custom kernel for realme C2 (RMX1941), based on Linux 4.9.242, MediaTek MT6765 (Helio P35).

## Specifications

- **Base**: Linux 4.9.242
- **Platform**: MediaTek MT6765 (Helio P35)
- **Defconfig**: `RMX1941-lunarch_defconfig`
- **Root solution**: ReSukiSU v4.2.0-rc1, Manual Hook mode (`CONFIG_KPROBES` disabled)
- **Container support**: Namespaces + cgroups (Docker-ready), OverlayFS, `CONFIG_BLK_CGROUP`, `CONFIG_BINFMT_MISC`
- **Networking**: Full iptables/ip6tables, IPsec, PPP/VPN, Bridge, WireGuard (built-in module)
- **Filesystem**: ext4, exFAT, NTFS (rw), FUSE, OverlayFS
- **Security**: SELinux (develop mode, togglable Permissive/Enforcing)

## Building

### Toolchain

```bash
sudo apt-get install -y clang llvm lld gcc-aarch64-linux-gnu gcc-arm-linux-gnueabi make bc bison flex libssl-dev libncurses-dev
```

Device tree generation requires Python 2 (`tools/dct/DrvGen.py`). On systems without a `python2` package (e.g. Ubuntu 24.04+), build Python 2.7.18 from source and symlink it as `python2`.

### Build

```bash
chmod +x build
./build
```

Output: `Image.gz-dtb` (kernel image + merged device tree blob), located under the build output directory.

## ReSukiSU Integration

This kernel uses **manual hook** mode (not kprobe) for ReSukiSU, following community guidance for older non-GKI kernels (4.4/4.9) where kprobe/ftrace infrastructure is historically unstable on MediaTek BSPs.

### Hook points

| Hook | File |
|---|---|
| `ksu_handle_execveat` | `fs/exec.c` |
| `ksu_handle_faccessat` | `fs/open.c` |
| `ksu_handle_stat` | `fs/stat.c` |
| `ksu_handle_newfstat_ret` | `fs/stat.c` |
| `ksu_handle_fstat64_ret` | `fs/stat.c` |
| `ksu_handle_sys_reboot` | `kernel/reboot.c` |
| `ksu_handle_input_handle_event` | `drivers/input/input.c` |

The build includes a self-check (`manual_hook_check.mk`) that fails early if any required hook is missing or mismatched.

### Defconfig additions for ReSukiSU

```
CONFIG_KSU=y
CONFIG_KSU_MANUAL_HOOK=y
CONFIG_KSU_MANUAL_HOOK_AUTO_SETUID_HOOK=y
CONFIG_KALLSYMS_ALL=y
```

## Flashing

1. Install a custom recovery that supports dynamic partitions (LineageOS Recovery / TWRP for RMX1941, Android 13).
2. Backup the boot partition (`boot.emmc.win` + checksum) as a rollback path.
3. Flash the AnyKernel3 zip via recovery.
4. Reboot.
5. Install the ReSukiSU manager APK matching the kernel driver version code to manage root access and modules.

## Credit

- Kernel & maintenance: Kinkatkut
- Build host: KinkatHost
- Root solution: [ReSukiSU](https://github.com/ReSukiSU/ReSukiSU)
