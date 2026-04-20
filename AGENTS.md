# AGENTS.md

This is a config repository for building the Xiaomi Surya kernel with Droidspaces container support. The actual kernel source is cloned during CI from `https://github.com/baikalos/android_kernel_xiaomi_surya.git`.

## Repository Contents

- `droidspaces.config` - Kernel config fragment enabling namespaces, cgroups, seccomp, netfilter, and other containerization features for Droidspaces
- `.github/workflows/build-surya.yml` - CI workflow that builds the kernel

## Build Process (from CI)

1. Clone kernel source: `git clone --depth=1 https://github.com/baikalos/android_kernel_xiaomi_surya.git`
2. Apply defconfig: `make O=out surya_defconfig`
3. Merge droidspaces config: `ARCH=arm64 scripts/kconfig/merge_config.sh -O out out/.config ./droidspaces.config`
4. Apply olddefconfig: `make O=out ARCH=arm64 olddefconfig`
5. Verify merged config: `grep` for `CONFIG_SYSVIPC`, `CONFIG_POSIX_MQUEUE`, `CONFIG_PID_NS`, `CONFIG_IPC_NS`, and `CONFIG_DEVTMPFS` in `out/.config`
6. Build: `make -j$(nproc --all) O=out ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=clang HOSTCC=clang HOSTCXX=clang++ LLVM=1 LLVM_IAS=1 KCFLAGS="-Wno-error" Image.gz`

## Notes

- Build uses Clang/LLVM and keeps `KCFLAGS="-Wno-error"` for a release-style compile
- Build target: `Image.gz` only (dtbs skipped due to broken DTS overlay files in both BaikalOS and LineageOS sources)
- Build output: `out/arch/arm64/boot/Image.gz`
- Workflow records the exact BaikalOS source HEAD in `baikal-source-commit.txt`
- Workflow uploads the merged final config as `final-surya.config`
- Target device: Xiaomi Surya (Poco X3 / Redmi Note 9 Pro)
- Droidspaces non-GKI patches applied: xt_qtaguid panic fix + cgroup file prefix fix
- LineageOS source (lineage-21) also tested and works; switch clone URL to try alternate sources
