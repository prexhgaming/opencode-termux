# Building OpenCode Termux ARM 32 Locally

## Overview

Building ARM 32 architecture binaries directly on Termux is recommended when GitHub Actions runners don't support this architecture or face compatibility issues. This guide provides step-by-step instructions for setting up your local Termux environment and building the project.

---

## Prerequisites

### System Requirements

- **Device**: ARM 32-bit Android device or emulator
- **Termux**: Latest version from [F-Droid](https://f-droid.org/packages/com.termux/) or [GitHub Releases](https://github.com/termux/termux-app/releases)
- **Storage**: At least 2-3 GB free space for dependencies and builds
- **RAM**: Minimum 2 GB recommended

### Permissions

After installing Termux:
```bash
termux-setup-storage
```

This grants Termux access to your device's storage.

---

## Step 1: Update Termux

Update the package repository:

```bash
pkg update
pkg upgrade -y
```

---

## Step 2: Install Required Build Tools

### Essential Tools

```bash
pkg install -y \
    build-essential \
    git \
    cmake \
    pkg-config \
    clang \
    binutils
```

### Language-Specific Tools

Depending on your project, install relevant toolchains:

**For C/C++ Projects:**
```bash
pkg install -y \
    gcc \
    g++ \
    gdb \
    make
```

**For Node.js Projects:**
```bash
pkg install -y nodejs npm
npm install -g npm
```

**For Python Projects:**
```bash
pkg install -y python python-pip
pip install --upgrade pip
```

**For Rust Projects:**
```bash
pkg install -y rustup
rustup default stable
```

### Additional Dependencies

Check if your project requires:
```bash
pkg install -y \
    openssl-dev \
    curl-dev \
    zlib-dev \
    readline-dev
```

---

## Step 3: Clone the Repository

```bash
cd ~
git clone https://github.com/prexhgaming/opencode-termux.git
cd opencode-termux
```

---

## Step 4: Build the Project

### Option A: Using CMake (Recommended for C/C++)

```bash
mkdir -p build
cd build
cmake ..
make -j$(nproc)
```

### Option B: Using Make

```bash
make -j$(nproc)
```

### Option C: Using Custom Build Script

If a build script exists in the repository:

```bash
bash ./build.sh
# or
python build.py
```

Check `README.md` in the repository for specific build instructions.

---

## Step 5: Verify the Build

After compilation completes:

```bash
# Check if binaries were created
ls -la build/  # or appropriate output directory

# Run basic tests if available
make test
# or
./build/your-binary-name --version
```

---

## Step 6: Create Build Artifacts

### For CI/CD Integration

Create a tar archive of the compiled binaries:

```bash
cd ~/opencode-termux
tar -czf opencode-termux-arm32-build.tar.gz build/
```

### Upload to Storage

To move artifacts to your device storage:

```bash
cp opencode-termux-arm32-build.tar.gz ~/storage/downloads/
```

---

## Troubleshooting

### Issue: Out of Memory

**Solution:**
- Build with fewer parallel jobs:
  ```bash
  make -j2
  ```
- Kill unnecessary processes:
  ```bash
  ps aux | grep -v grep | awk '{print $2}' | xargs kill -9
  ```

### Issue: Missing Dependencies

**Solution:**
- Search for available packages:
  ```bash
  pkg search <package-name>
  ```
- Install missing development headers:
  ```bash
  pkg install -y <package-name>-dev
  ```

### Issue: Compilation Errors

**Solution:**
1. Check the error message carefully
2. Ensure all dependencies are installed
3. Try cleaning the build directory:
   ```bash
   rm -rf build/
   mkdir build
   cd build
   cmake ..
   make
   ```

### Issue: Permission Denied

**Solution:**
```bash
# Grant execution permissions
chmod +x ./build/binary-name

# Or make entire directory executable
chmod -R +x ./build/
```

---

## Performance Tips

1. **Use Release Build:**
   ```bash
   cmake -DCMAKE_BUILD_TYPE=Release ..
   make
   ```

2. **Parallel Compilation:**
   ```bash
   make -j$(nproc)
   ```
   This uses all available CPU cores.

3. **Ccache for Faster Rebuilds:**
   ```bash
   pkg install -y ccache
   export CC="ccache gcc"
   export CXX="ccache g++"
   ```

4. **Monitor Resources:**
   ```bash
   # Install htop for system monitoring
   pkg install -y htop
   htop
   ```

---

## Continuous Building

For development workflows, automate rebuilds:

```bash
#!/bin/bash
# save as rebuild.sh

cd ~/opencode-termux
while true; do
    git pull
    rm -rf build/
    mkdir build
    cd build
    cmake ..
    make -j$(nproc)
    cd ..
    echo "Build completed at $(date)"
    sleep 3600  # Rebuild every hour
done
```

Run in background:
```bash
nohup bash rebuild.sh > build.log 2>&1 &
```

---

## Next Steps

### Publish Builds

1. **GitHub Releases:**
   - Upload tar archive to GitHub Releases
   - Create CI/CD workflow for automated uploads

2. **Testing:**
   - Run unit tests: `make test`
   - Test on multiple ARM 32 devices if possible

3. **Documentation:**
   - Document any device-specific requirements
   - Note successful build configurations

---

## Additional Resources

- [Termux Wiki](https://wiki.termux.com/)
- [Termux Packages](https://packages.termux.dev/)
- [CMake Documentation](https://cmake.org/documentation/)
- [GNU Make Manual](https://www.gnu.org/software/make/manual/)

---

## Support

For issues specific to OpenCode Termux, check:
- Repository Issues: [GitHub Issues](https://github.com/prexhgaming/opencode-termux/issues)
- Parent Project: [Hope2333/opencode-termux](https://github.com/Hope2333/opencode-termux)

---

*Last updated: October 2026*
