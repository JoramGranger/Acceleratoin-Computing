# Nvidia A100 GPU Testing
### 1. Install Monitoring tools
```
dnf install -y epel-release
dnf install -y htop stress-ng nvtop   # nvtop for nice GPU monitoring (like htop for GPUs)
```

### 2. Setup NVIDIA DCGM (Data Center GPU Manager) Diagnostics
#### 2.1 Install DCGM on Rocky 9 (from NVIDIA repo — works great on RHEL9 clones): Add DCGM repo (if not already; check /etc/yum.repos.d/)
```
dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel9/x86_64/cuda-rhel9.repo
```

#### 2.2 check cuda version for installed GPU
```
nvidia-smi
```
#### 2.3 search
```
dnf search datacenter-gpu-manager
```
#### 2.4 Install DCGM
```
dnf install -y datacenter-gpu-manager-4-cuda13
```

#### 2.5 Start the service
```
systemctl daemon-reload
systemctl enable --now nvidia-dcgm
systemctl status nvidia-dcgm   # Should show active (running) with no errors
```
#### or
```
systemctl restart nvidia-dcgm
systemctl status nvidia-dcgm   # Check for "Active: active (running)" and no errors in recent logs
journalctl -u nvidia-dcgm -n 50   # Look for startup messages or errors
```

#### 2.6 run discovery (waif for some seconds) and verify DCGM version
```
dcgmi --version   # If command available; expect 4.3.x or similar
dcgmi discovery -l   # Should now list your GPU(s): e.g., GPU ID 0: NVIDIA A100 80GB PCIe
```
###  3 Run stress tests:

#### 3.1 Quick level 1–2 (basic functional):Bash

##### 3.1  hardware test
```
dcgmi diag -r 1   # Level 1: quick checks
```

##### 3.2  hardware test
```
dcgmi diag -r 2   # Level 2: more thorough
```

##### 3.3 Full stress (level 3: includes targeted stress plugin for power, thermal, throughput, memory bandwidth — run for 10–60+ mins).
```
dcgmi diag -r 3 -p diagnostic.test_duration=600.0   
# 10 minutes; increase to 3600 for 1 hour
```

#### Or targeted stress only (constant high load):Bash
```
dcgmi diag -r 3 -p targeted_stress.test_duration=1800.0 targeted_stress.temperature_max=85.0
```

### 4. Simple Compute Stress: gpu-burn (Great for Quick Power/Compute Load)
#### 4.1 Install
```
git clone https://github.com/wilicc/gpu-burn
cd gpu-burn
make
```

#### 4.2 Run (stresses all GPUs, adjust -d for duration in seconds)
```
./gpu_burn 3600   # 1 hour; use & for background if needed
```