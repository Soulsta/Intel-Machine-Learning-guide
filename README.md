# Intel Arc A770 Dual GPU Setup for LLM Inference

## Hardware Configuration

- CPU: Intel i5-12600KF (10 cores: 6P + 4E)
- RAM: 112GB DDR4/DDR5
- GPU: 2x Intel Arc A770 16GB (32GB VRAM total)
- Storage: 240GB NVMe SSD (mounted at /mnt/nvme)
- OS: Ubuntu 22.04 LTS

## Setup Overview

This guide documents the complete setup process for running large language models on dual Intel Arc A770 GPUs using llama.cpp with Intel SYCL support.

---

## Step 1: NVMe Storage Setup

### Mount and Format NVMe Drive
```bash
# Check available disks
lsblk

# Format partition as ext4 (if needed)
sudo mkfs.ext4 /dev/nvme0n1p1

# Create mount point
sudo mkdir -p /mnt/nvme

# Mount the drive
sudo mount /dev/nvme0n1p1 /mnt/nvme

# Set ownership
sudo chown -R $USER:$USER /mnt/nvme
sudo chmod 755 /mnt/nvme
```

### Configure Auto-Mount
```bash
# Get UUID of the partition
sudo blkid /dev/nvme0n1p1

# Edit fstab
sudo nano /etc/fstab

# Add this line (replace UUID with your actual UUID):
UUID=your-uuid-here  /mnt/nvme  ext4  defaults,nofail  0  2

# Test auto-mount
sudo umount /mnt/nvme
sudo mount -a
df -h /mnt/nvme
```

### Add Label (Optional)
```bash
sudo e2label /dev/nvme0n1p1 AI_Storage
```

---

## Step 2: Intel GPU Driver Installation

### Add Intel GPU Repository
```bash
# Add Intel graphics repository key
wget -qO - https://repositories.intel.com/gpu/intel-graphics.key | \
  sudo gpg --dearmor --output /usr/share/keyrings/intel-graphics.gpg

# Add repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/intel-graphics.gpg] https://repositories.intel.com/gpu/ubuntu jammy client" | \
  sudo tee /etc/apt/sources.list.d/intel-gpu-jammy.list

sudo apt update
```

### Install GPU Drivers and Runtime
```bash
sudo apt install -y \
  intel-opencl-icd \
  intel-level-zero-gpu \
  level-zero \
  intel-media-va-driver-non-free \
  libmfx1 \
  libmfxgen1 \
  libvpl2 \
  clinfo \
  intel-gpu-tools
```

### Verify GPU Detection
```bash
# Check OpenCL devices
clinfo | grep "Device Name"

# Should show both Arc A770 Graphics

# Check GPU device nodes
ls -la /dev/dri/

# Should show renderD128 and renderD129
```

---
