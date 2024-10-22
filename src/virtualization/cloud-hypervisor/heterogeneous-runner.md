# Heterogeneous Runner

RISC-V SoC with H ext. and AIA equipped is not available at the time of writing
(Oct 22nd 2024), so I conduct my development of RISC-V virtualization software
stack on virtual machines emulated by QEMU (v9.1.0), on x86_64 hosts.

## Setup

### Host

Output of `neofetch`:

```
❯ neofetch
            .-/+oossssoo+/-.               root@cloud-hypervisor-riscv64
        `:+ssssssssssssssssss+:`           -----------------------------
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 24.04.1 LTS x86_64
    .ossssssssssssssssssdMMMNysssso.       Host: MS-7E16 1.0
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Kernel: 6.8.0-47-generic
  +ssssssssshmydMMMMMMMNddddyssssssss+     Uptime: 3 days, 21 hours, 26 mins
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Packages: 909 (dpkg)
.ssssssssdMMMNhsssssssssshNMMMdssssssss.   Shell: bash 5.2.21
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Terminal: /dev/pts/0
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   CPU: AMD Ryzen 9 9950X (32) @ 7.487GHz
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   GPU: AMD ATI 15:00.0 Device 13c0
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Memory: 3444MiB / 61876MiB
.ssssssssdMMMNhsssssssssshNMMMdssssssss.
 /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/
  +sssssssssdmydMMMMMMMMddddyssssssss+
   /ssssssssssshdmNNNNmyNMMMMhssssss/
    .ossssssssssssssssssdMMMNysssso.
      -+sssssssssssssssssyyyssss+-
        `:+ssssssssssssssssss+:`
            .-/+oossssoo+/-.
```

I'm using Ubuntu 24.04 LTS so I guess there would not be any serious blockers if
you use this version or any version above that. And I suggest you to pick a CPU
with high frequency, since we are going to emulate a heterogeneous virtual
machine, you are going to need it :)

#### QEMU

Generally we need to check against the upstream QEMU for features/fixes we need,
since RISC-V is still under heavy development, bugs like passing the wrong
`hart id` could happen, and features like `aia` or `iommu` which we very much
interested could develop, just visit https://github.com/qemu/qemu to find out
which version meets your requirement.

Below is the steps took to install QEMU manually:

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

#### OPENSBI & UBOOT

Besides QEMU, we need `opensbi` and `uboot` to launch an Ubuntu RISC-V virtual
machine. These two could be installed through command:

```sh
sudo apt-get install opensbi u-boot-qemu
```

#### RISC-V Image

I'm using a pre-installed RISC-V Ubuntu server image, which could be downloaded:

```sh
# Download Ubuntu 24.04.1 LTS
wget https://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04.1-preinstalled-server-riscv64.img.xz
# Extract image
xz -dk ubuntu-24.04.1-preinstalled-server-riscv64.img.xz
```
