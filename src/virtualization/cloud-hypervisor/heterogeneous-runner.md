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
# Resize the image
# The image defaults to 5 GiB, couldn't even fit in a kernel source code
qemu-img resize -f raw ubuntu-24.04.1-preinstalled-server-riscv64.img +45G
```

#### Start Script

The QEMU command used to start a RISC-V guest is normally lengthy, let's
simplify it by writing them into a script named `start-riscv64.sh`:

```bash
# Change memory and cores that fits in your host
qemu-system-riscv64 \
    -machine virt \
    -nographic \
    -m 96G \
    -smp 16 \
    -bios /usr/lib/riscv64-linux-gnu/opensbi/generic/fw_jump.bin \
    -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
    -device virtio-net-device,netdev=eth0 \
    -netdev user,id=eth0,hostfwd=tcp::12055-:22 \
    -device virtio-rng-pci \
    -drive file=$1,format=raw,if=virtio \
```

This script is sufficient to boot a fully emulated RISC-V VM, but insufficient
to be the development environment because its kernel version does not support
AIA. With this script in place, you can start VM with command:

```sh
bash start-riscv64.sh
```

### Guest

You may now login into VM booted with user name and password being both
`ubuntu`. After you have logged in, you will need to execute
`sudo systemctl enable --now ssh` to make it standby, then you can ssh into this
VM with `ssh ubuntu@localhost -p 12055`.

#### Kernel

As mentioned previously, the kernel shipped with Ubuntu 24.04.1 LTS is
insufficient to develop type II hypervisors meet RVA23 standards. We will need
to manually replace the kernel from upstream kernel source:

```sh
sudo apt update
sudo DEBIAN_FRONTEND="noninteractive" apt-get install --no-install-recommends -y \
    git python3 python3-pip ninja-build build-essential pkg-config curl bc jq \
    libslirp-dev libfdt-dev libglib2.0-dev libssl-dev libpixman-1-dev \
    flex bison

git clone --depth 1 --branch v6.11 https://github.com/torvalds/linux.git
pushd linux
make defconfig
# Enable kvm module instead of inserting manually
sed -i "s|^CONFIG_KVM=.*|CONFIG_KVM=y|g" .config
# Build kernel
make -j$(nproc)
# Install kernel
sudo make headers_install
sudo make modules_install
# You might need to repeat this step, it somehow fails while installing for the
# first time, retry should fix it. (PRs are welcome to explain why this happens)
sudo make install
sudo update-grub
popd
```

After kernel is properly installed, you may power it off and start it again with
AIA enabled.

#### Boot with AIA

The new kernel should be able to boot with AIA in place, change your start
script like this:

```bash
# Change memory and cores that fits in your host
qemu-system-riscv64 \
    -machine virt,aia=aplic-imsic \
    -nographic \
    -m 96G \
    -smp 16 \
    -bios /usr/lib/riscv64-linux-gnu/opensbi/generic/fw_jump.bin \
    -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
    -device virtio-net-device,netdev=eth0 \
    -netdev user,id=eth0,hostfwd=tcp::12055-:22 \
    -device virtio-rng-pci \
    -drive file=$1,format=raw,if=virtio \
```

Now the VM is ready for type II hypervisor development.
