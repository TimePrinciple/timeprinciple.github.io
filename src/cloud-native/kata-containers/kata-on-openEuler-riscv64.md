# Kata-Containers On openEuler RISC-V 64-bit

If you are verifying kata-containers' availability on openEuler RISC-V 64-bit
system, using VMM except for QEMU, you will need a QEMU with version greater
than or equal to v9.0.2 for that purpose.

## Prepare QEMU

```sh
# Install dependencies to build and run QEMU
DEBIAN_FRONTEND="noninteractive" apt-get install --no-install-recommends -y \
    git python3 python3-pip ninja-build build-essential pkg-config curl bc jq \
    libslirp-dev libfdt-dev libglib2.0-dev libssl-dev libpixman-1-dev \
    flex bison

# Clone QEMU source code repository
git clone --depth 1 --branch v9.1.0 https://gitlab.com/qemu-project/qemu.git

pushd qemu
# Prepare directory to for QEMU to be installed
mkdir -p /opt/qemu
# Build and install QEMU
./configure --prefix=/opt/qemu && make -j$(nproc)
# This requires root privilege
sudo make install
popd
# Clean up
rm -rf qemu

# Add line below to your .bashrc (.zshrc or scripts in profile.d/ whatever)
export PATH=$PATH:/opt/qemu/bin

. .bashrc
```
