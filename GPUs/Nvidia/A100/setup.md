# Nvidia A100 setup with Rocky Linux 9.7

### 1. Update system + install prerequisites
#### 1.1 install prerequisites
```
dnf update -y
dnf install -y epel-release
dnf install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r) gcc make dkms acpid
```

### 2. Blacklist nouveau
#### 2.1 blacklist
```
cat <<EOF > /etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
EOF
```

#### 2.2 dracut
```
dracut --force --regenerate-all
```


#### 2.3 Add to GRUB for safety
```
sed -i 's/GRUB_CMDLINE_LINUX="/GRUB_CMDLINE_LINUX="nomodeset rd.driver.blacklist=nouveau /' /etc/default/grub
grub2-mkconfig -o /boot/grub2/grub.cfg   # or /boot/efi/... if EFI boot
```

#### 2.4 reboot
```
reboot +0
```

#### 2.5 After reboot, verify:
```
lsmod | grep nouveau   # must be empty
```

### 3. Add NVIDIA CUDA network repo
#### 3.1 add repo
```
dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel9/x86_64/cuda-rhel9.repo
dnf clean all
dnf makecache
```

### 4. enable the open stream

#### 4.1 enable
```
dnf module list nvidia-driver   # Run this to see actual available streams/modules
```

#### 4.2 Look for something like: nvidia-driver open-dkms [d] or nvidia-driver:latest-open etc.
#### If you see open-dkms or similar, enable it:
```
dnf module enable -y nvidia-driver:open-dkms
```

##### OR if it's listed as open (no -dkms suffix):
```
dnf module enable -y nvidia-driver:open
```

#### 4.3 Install driver + minimal CUDA runtime (headless/compute only) 
```
dnf module install -y nvidia-driver:latest-open-dkms
```

#### 4.4 Refresh metadata first if needed:Bash
```
dnf clean all
dnf makecache
```

#### 4.5 reboot
```
reboot
```

#### 4.6 verify after reboot
```
nvidia-smi
```

#### 4.7 Persistence is off (normal for now; enable with if doing long runs).
```
nvidia-smi -pm 1
```

### 5. Install Full CUDA Toolkit (essential for ML frameworks like PyTorch/TensorFlow, RAPIDS, Parabricks, nvcc compilation, etc.):

#### 5.1 install cuda
```
dnf install -y cuda-toolkit
```

#### 5.2 Add paths system-wide:
```
cat <<EOF > /etc/profile.d/cuda.sh
export PATH=/usr/local/cuda/bin:\$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:\$LD_LIBRARY_PATH
EOF
```

#### 5.3 Test:
```
source /etc/profile.d/cuda.sh
nvcc --version
```
