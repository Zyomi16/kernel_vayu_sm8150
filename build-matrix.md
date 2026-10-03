# Ma trận 20 bản kernel vayu (non-GKI 4.14.357+17)

Cây: `/root/kern`, root xác minh `Makefile:2-5` (`VERSION=4 PATCHLEVEL=14
SUBLEVEL=357 EXTRAVERSION=+17`), `make kernelversion` → `4.14.357+17`.
Không có `common/`, không `BUILD.bazel` → non-GKI, kernel root là gốc repo.
Defconfig: `arch/arm64/configs/vayu_defconfig` (168,188 B, 1,861 `CONFIG_`).
Toolchain ghim: `clang-11 lld-11 llvm-11 gcc-11-aarch64-linux-gnu`
trên `ubuntu-22.04` (`matrix.yml`, apt list đầy đủ + `clang --version` in log).
Công thức Clang+GCC (`LLVM=1` + `CROSS_COMPILE` + `CLANG_TRIPLE`) theo
`Android-Kernel-Tutorials/README.md:401-435` (cây 4.14 cần cả hai;
bỏ `CROSS_COMPILE` chỉ đúng cho 5.10+/AOSP).

Merge: `vayu_defconfig` → `yomi16.config` → fragment root → fragment matrix
(`merge_config.sh -m`) → `olddefconfig`. Không sửa defconfig OEM.
Nhóm cấm (không đụng): `MODVERSIONS/MODULE_SIG_FORCE/SELINUX/DM_VERITY/
OVERLAY_FS/CGROUP*/NET_CLS_*/NET_SCH_*/NAMESPACES/EFI_PARTITION`.
`yomi16.config` có `CONFIG_IPC_NS` — đây KHÔNG phải symbol `CONFIG_NAMESPACES`
(base defconfig đã có `NAMESPACES=y`, không đổi), nên không vi phạm.

LTO trên 4.14: `LTO_CLANG` (`arch/Kconfig:657`) và `THINLTO` (`arch/Kconfig:641`)
đều tồn tại → không có mục `[không có option]`. `LTO_CLANG_BOLT/MLGO`, PGO:
quét toàn bộ Kconfig không có symbol nào → không đưa vào ma trận.
`-O3`: không có Kconfig option (chỉ `CC_OPTIMIZE_FOR_PERFORMANCE=-O2` và
`CC_OPTIMIZE_FOR_SIZE=-Os`, `init/Kconfig:1170-1177`) → ma trận chỉ dùng O2/Os.
`ld-name=lld` bắt buộc khi LTO bật (`Makefile:707`, LD rỗng nếu thiếu —
xác minh bằng probe run 37084024479).

## Chiều ma trận

- root: `none` (4) / `resukisu` (8) / `sukisu-ultra` (8) = 20
- susfs: chỉ `resukisu` (driver SukiSU không có `KSU_SUSFS` — đã grep Kconfig)
- kpm: chỉ `sukisu-ultra` (`KPM` là symbol SukiSU)
- lto: on (`LTO_CLANG+THINLTO`) / off (`LTO_NONE`, fragment `m/lto-off.config`)
- opt: O2 (mặc định) / Os (`m/opt-size.config`)

| # | Tên bản | Fragment root | LTO | OPT | Swap | Trạng thái lúc lập |
|---|---------|---------------|-----|-----|------|--------------------|
| 01 | none-lto-o2 | none | on | O2 | - | chưa build |
| 02 | none-nolto-o2 | none | off | O2 | - | chưa build |
| 03 | none-lto-os | none | on | Os | - | chưa build |
| 04 | none-nolto-os | none | off | Os | - | chưa build |
| 05 | resukisu-lto-o2 | resukisu | on | O2 | - | ✅ xanh nhiều run (= build hiện tại) |
| 06 | resukisu-nolto-o2 | resukisu | off | O2 | - | chưa build |
| 07 | resukisu-lto-os | resukisu | on | Os | - | chưa build |
| 08 | resukisu-nolto-os | resukisu | off | Os | - | chưa build |
| 09 | resukisu-susfs-lto-o2 | resukisu-susfs | on | O2 | - | đang build run 37106693951 |
| 10 | resukisu-susfs-nolto-o2 | resukisu-susfs | off | O2 | - | chờ 09 |
| 11 | resukisu-susfs-lto-os | resukisu-susfs | on | Os | - | chờ 09 |
| 12 | resukisu-susfs-nolto-os | resukisu-susfs | off | Os | - | chờ 09 |
| 13 | sukisu-lto-o2 | sukisu-ultra | on | O2 | sukisu | chưa từng compile (fail oan bước grep, đã sửa) |
| 14 | sukisu-nolto-o2 | sukisu-ultra | off | O2 | sukisu | như 13 |
| 15 | sukisu-lto-os | sukisu-ultra | on | Os | sukisu | như 13 |
| 16 | sukisu-nolto-os | sukisu-ultra | off | Os | sukisu | như 13 |
| 17 | sukisu-kpm-lto-o2 | sukisu-ultra-kpm | on | O2 | sukisu | thiếu `uapi/` đã sửa, chưa thử lại |
| 18 | sukisu-kpm-nolto-o2 | sukisu-ultra-kpm | off | O2 | sukisu | như 17 |
| 19 | sukisu-kpm-lto-os | sukisu-ultra-kpm | on | Os | sukisu | như 17 |
| 20 | sukisu-kpm-nolto-os | sukisu-ultra-kpm | off | Os | sukisu | như 17 |

## Lệch khỏi prompt (ghi rõ lý do)

- Không cache `~/toolchains` + ccache: toolchain từ apt (~1-2 phút),
  `CC` bị Makefile gán cứng (`CC = clang`, `Makefile:392`) nên ccache cần
  wrapper PATH — rủi ro phát hiện sai toolchain, lợi ít hơn hại.
- Không `-O3`: không có Kconfig option, ép bằng `KCFLAGS` là đường chưa verify.
- Không build GCC-only: công thức đã verify là Clang+GCC; GCC-only là biến
  mới cần verify riêng, ngoài ngân sách 10h.
