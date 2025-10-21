# RHEL for ARM64 on WSL2

> **Note**: This seems to be the only way to get RHEL running on WSL for ARM devices, as Red Hat doesn't provide official ARM WSL images.

## Prerequisites
- Windows 11 ARM64 device
- WSL2 installed
- Another WSL2 distribution with Docker installed


## Configuration
- Replace `<WINDOWSUSER>` with your Windows username
- Replace `<DISTRONAME>` with your desired WSL distribution name
- Replace `<USERNAME>` with your desired Linux username
- Replace `<ORG_ID>` and `<KEY>` with your Red Hat credentials

## Notes
- This method uses Red Hat Universal Base Image (UBI) 10
- Systemd is enabled for proper service management
- The distribution will behave like a standard RHEL system

## Credit
Reddit user [greengorych](https://www.reddit.com/user/greengorych/) provides the above explanation in this [thread](https://www.reddit.com/r/wsl2/comments/1lxczzc/red_hat_wsl2_for_arm64_devices/)

## Installation

### 1. Download RHEL Container Image
In your existing WSL2 distribution:
```sh
sudo docker pull redhat/ubi10-init:latest
sudo docker create --name ubi10 redhat/ubi10-init:latest
sudo docker export ubi10 -o ubi10-rootfs.tar
sudo mkdir -p /mnt/c/Users/<WINDOWSUSER>/Downloads/wsl
sudo mv ubi10-rootfs.tar /mnt/c/Users/<WINDOWSUSER>/Downloads/wsl
```

### 2. Import to WSL2
In PowerShell:
```ps
wsl --import <DISTRONAME> "C:\Users\<WINDOWSUSER>\AppData\Local\Packages\<DISTRONAME>" .\ubi10-rootfs.tar
```

### 3. Configure User and WSL Settings
In PowerShell:
```ps
wsl -d <DISTRONAME>
```

In the RHEL environment:
```sh
# Install sudo
dnf install sudo -y

# Create user
useradd <USERNAME>
passwd <USERNAME>
usermod -aG wheel <USERNAME>

# Configure WSL
echo -e "[boot]\nsystemd=true\n[network]\nhostname=<DISTRONAME>\ngenerateHosts=false\n[user]\ndefault=<USERNAME>" | sudo tee /etc/wsl.conf

exit
```

### 4. Restart WSL2
```ps
wsl --terminate <DISTRONAME>
```

### 5. Register with Red Hat
1. Sign up at [Red Hat Developer](https://developers.redhat.com/)
2. Get your activation key [here](https://console.redhat.com/insights/connector/activation-keys) and organization ID from your profile

3. In PowerShell, open your WSL2 RHEL instance:
```ps
wsl -d <DISTRONAME>
```

4. In the RHEL environment:
```sh
sudo subscription-manager register --org <ORG_ID> --activationkey <KEY>
```
5. Output should show:
```
The system has been registered with ID: <HASH>
The registered system name is: <DISTRONAME>
```

### 6. Tidy Up

After verifying your WSL distribution is set up and registered, you may want to delete the `Downloads\wsl\`

This step is optional but recommended to keep your workspace clean.

