# Local ARMv7/ARM32 build guide for this repo

## Important reality check

This repository is not a generic C/C++ project that builds with a single `cmake .. && make` flow. The actual source tree is a Termux packaging + binary transplant pipeline for Android/Bionic.

The repo itself explicitly contains:

- patching for Android support in `patches/0001-android-support.patch`
- Termux CWD fixes in `patches/0002-bun-termux-cwd-fix.patch`
- transplant/build tooling under `tools/transplant/`
- Make targets that produce the patched native binary, including:
  - `make transplant VER=<version>`
  - `make family-native VER=<version>`
  - `make family-compressed VER=<version>`
  - `make seccomp-harden VER=<version>`

This is the real path the repo uses for native Android builds.

At the same time, the repo docs state that ARMv7/arm32 work is deferred/non-mainline:

- `docs/README.md` says: `arm32 and broader armv7 portability implementation work is deferred/non-mainline unless explicitly promoted.`

So: local building is valid if you are doing a targeted experimental/fallback armv7/arm32 build, but it is not the standard mainline packaging flow.

---

## What the repo is actually building

The repo's mainline path is the Android/Bionic transplant pipeline, not a straight `make` from the upstream source tree.

Relevant files:

- `Makefile` — orchestrates `transplant`, `libopentui`, `seccomp-harden`, `transplant-upx`
- `patches/README.md` — explains the Android patch flow
- `docs/ci-prebuild-armv7.md` — documents armv7 cross-build / native fallback attempts
- `docs/armv7-native-runner-setup.md` — explains self-hosted armv7 runner setup for CI fallback

This means the binary needing patching is the upstream OpenCode runtime binary after it is transplanted onto Android/Bionic, not a generic binary produced by a trivial CMake build.

---

## Recommended local workflow for ARMv7/ARM32

### 1) Use a real ARMv7/arm32 host if possible

For ARMv7/arm32, the repo expects a real native environment or a self-hosted runner, per `docs/armv7-native-runner-setup.md`.

Check architecture:

```bash
uname -m
file /bin/sh
```

Expected output should look like:

```bash
armv7l
ELF 32-bit LSB executable, ARM ...
```

If you are on Android/Termux, the environment is only a realistic fallback for experimenting, not the canonical release path.

---

### 2) Install build dependencies

In Termux:

```bash
pkg update && pkg upgrade -y
pkg install -y git curl ca-certificates build-essential cmake ninja python pkg-config clang llvm rustup nodejs npm
```

Optional but useful:

```bash
pkg install -y ccache upx
```

---

### 3) Clone repo and inspect the actual build targets

```bash
git clone https://github.com/prexhgaming/opencode-termux.git
cd opencode-termux
```

Then inspect the Makefile targets:

```bash
make help | sed -n '1,120p'
```

The important targets are:

```bash
make transplant VER=<version>
make family-native VER=<version>
make family-compressed VER=<version>
```

---

### 4) Build the native Android binary using the repo's transplant pipeline

For a specific version (example):

```bash
make transplant VER=2.0.0
```

This is the key repo-native build path. It performs the transplant, libopentui patching, and seccomp hardening workflow.

For native packages after transplant:

```bash
make family-native VER=2.0.0
```

This produces packaging artifacts with the patched runtime.

If you are testing the compressed packaging line:

```bash
make family-compressed VER=2.0.0
```

---

### 5) Verify the result

Check generated artifacts:

```bash
find artifacts -maxdepth 4 -type f | sort | tail -n 50
```

Or package output:

```bash
find packing -maxdepth 4 -type f | sort | tail -n 50
```

Check the binary type:

```bash
file artifacts/transplant/<ver>/opencode-native
```

For ARMv7/arm32 the expected signal is:

```bash
ELF 32-bit LSB executable, ARM ...
```

---

## Extra ARMv7/arm32 debug workflow from the repo

The repo includes a dedicated fallback path for armv7 native builds:

- `docs/ci-prebuild-armv7.md`
- `docs/armv7-native-runner-setup.md`

That workflow is expressly designed for a real `armv7` self-hosted runner and checks:

```bash
uname -m
file /bin/sh
cmake --version
ninja --version
python3 --version
rustc --version || true
cargo --version || true
bun --version || true
```

If the native runner is absent, the repo treats armv7 as an attempt/fallback path, not a mainline supported package.

---

## When to build locally

Build locally only when:

- you are debugging a failing armv7/arm32 binary
- you need to validate a transplant patch on-device
- you are testing the experimental fallback path before CI/self-hosted deployment

Do not assume it is the normal release target. The repo explicitly labels ARMv7/arm32 as deferred/non-mainline.

---

## Bottom line

The repo does cover the binaries needing patching, but the correct process is not generic `cmake` compilation. It is the repo's custom Android/Bionic transplant flow:

```bash
make transplant VER=<version>
make family-native VER=<version>
```

If you need a true ARMv7/arm32 build, prefer a real ARMv7 host or a self-hosted runner, because the repo documents that as the supported fallback path.

---

## One-liner to try on Termux

```bash
pkg update && pkg upgrade -y && pkg install -y git curl ca-certificates build-essential cmake ninja python pkg-config clang llvm rustup nodejs npm && git clone https://github.com/prexhgaming/opencode-termux.git && cd opencode-termux && make transplant VER=2.0.0
```

If this fails, the likely cause is not “missing GCC”; it is that this repo's ARMv7/arm32 path is experimental and requires the actual transplant/patch toolchain described in the project docs.
