# NVIDIA Suspend Fix

This file explains a change made to your system on July 20, 2025, to fix an issue where the computer would immediately wake up after being suspended.

## The Problem

The system was failing to suspend properly due to an issue with the NVIDIA graphics driver. The driver was unable to preserve the video memory (VRAM) during the suspend process, which caused an error and prevented the system from staying in a suspended state.

The error message in the system logs was:
`NVRM: nvCheckOkFailedNoLog: Check failed: Out of memory [NV_ERR_NO_MEMORY]`

## The Fix

To resolve this, a new configuration file was created at:
`/etc/modprobe.d/nvidia-power-management.conf`

This file contains the following line:
```
options nvidia NVreg_PreserveVideoMemoryAllocations=0
```
This setting tells the NVIDIA driver **not** to save the contents of the graphics card's memory when the system suspends.

### Side Effects of this Fix

Because the video memory is not preserved, any applications that heavily use the GPU (such as games, video editing software, or 3D modeling tools) will likely crash or not render correctly when you resume the system. You will need to restart these applications after waking the computer. For most other desktop use, this is not an issue.

## How to Revert the Change

If you want to revert to the previous behavior, you can rename the configuration file to disable it.

**To disable the fix:**
Open a terminal and run the following command to back it up. This will prevent it from being loaded.
```bash
sudo mv /etc/modprobe.d/nvidia-power-management.conf /etc/modprobe.d/nvidia-power-management.conf.backup
```

**To re-enable the fix:**
If you want to re-apply the fix later, simply rename the file back:
```bash
sudo mv /etc/modprobe.d/nvidia-power-management.conf.backup /etc/modprobe.d/nvidia-power-management.conf
```

After running either command, you must **reboot your computer** for the change to take effect.
