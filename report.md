# Debian VM Installation via Serva PXE Boot - Implementation Report

## Project Overview
Successfully implemented a PXE (Preboot Execution Environment) boot server using Serva to install Debian Linux on a virtual machine over the network. This eliminates the need for physical installation media and enables automated network-based deployments.

## Prerequisites
- Set up [Serva](https://www.vercot.com/~serva/download.html) as a PXE boot server on Windows
- Download the [netboot](https://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/netboot/)  environment for Debian installation.
- Configure you VM. It's important to change the boot order and to put the network with the bridge adapter

## Network Boot Menu Screenshot

![PXE Boot Menu](images/Pasted%20image%2020250821122135.png)

## Implementation Steps

### Step 1: Environment Setup
**Location:** `C:\SERVA_ROOT\`
- Downloaded and extracted Serva PXE server software
- Verified directory structure with essential components:
  - `Serva64.exe` - Main application

- Open the app with admin privileges and enable TFTP and Proxy DHCP. This is the configuration that you should have for each connection

<div align="center">

<img src="images/dhcp.png" alt="Serva DHCP Configuration" width="400"/><br>
<sub><b>Figure 1:</b> Serva DHCP Server Configuration Panel</sub>

<br><br>

<img src="images/tftp.png" alt="Serva TFTP Configuration" width="400"/><br>
<sub><b>Figure 2:</b> Serva TFTP Server Configuration Panel</sub>

<br><br>

<img src="images/http.png" alt="Serva HTTP Configuration" width="400"/><br>
<sub><b>Figure 3:</b> Serva HTTP Server Configuration Panel</sub>

</div>

- Restart the application, and reopened with admin privileges. 

- Now you should have these folders. 

<div align="center">

<img src="images/serva-files.png" alt="Serva Directory Structure" width="500"/><br>
<sub><b>Figure 4:</b> SERVA_ROOT directory showing required files and folders for PXE boot</sub>

</div>


### Step 2: Debian Netboot Files Acquisition

**Source:** Official Debian repository
- Downloaded `netboot.tar.gz` from Debian mirrors

- Extracted netboot archive containing:
  - `linux` - Linux kernel (8,193,984 bytes)
  - `initrd.gz` - Initial RAM disk (40,697,786 bytes)
  - `pxelinux.0` - PXE bootloader
  - `pxelinux.cfg/` - Boot configuration directory
  - Supporting files for BIOS and UEFI boot



### Step 3: Directory Structure Configuration
**Target Structure:**
```
C:\SERVA_ROOT\
├── NWA_PXE\
│   └── Debian_12\
│       ├── debian-installer\
│       │   └── amd64\
│       │       ├── linux
│       │       ├── initrd.gz
│       │       ├── grub\
│       │       └── boot-screens\
│       ├── pxelinux.0
│       ├── pxelinux.cfg\
│       │   └── default
│       └── ServaAsset.inf (we will create this file in the next step)
```
I have to extract the files in WSL and later copy them in this folder because PowerShell does not recognize symbolic Links. 


### Step 4: ServaAsset.inf Configuration
Created asset information file following Serva v3.0 specifications:

```
ServaAsset.inf
;-Serva v3.0 Asset Information File 
;-Boot/Install:
;  Debian Debian 12 (bookworm), 10 (buster), Debian 9 (stretch), Debian  8 (jessie) 7 (wheezy) Network Install (requieres Internet)
;-Tested on:
;  netboot.tar.gz (amd64)
;    http://ftp.debian.org/debian/dists/bookworm/main/installer-amd64/current/images/netboot/netboot.tar.gz
;    http://ftp.debian.org/debian/dists/buster/main/installer-amd64/current/images/netboot/netboot.tar.gz
;    http://ftp.debian.org/debian/dists/stretch/main/installer-amd64/current/images/netboot/netboot.tar.gz
;    http://ftp.debian.org/debian/dists/jessie/main/installer-amd64/current/images/netboot/netboot.tar.gz
;    http://ftp.debian.org/debian/dists/wheezy/main/installer-amd64/current/images/netboot/netboot.tar.gz
;  netboot.tar.gz (i386)
;    http://ftp.debian.org/debian/dists/bookworm/main/installer-i386/current/images/netboot/netboot.tar.gz
;    http://ftp.debian.org/debian/dists/buster/main/installer-i386/current/images/netboot/netboot.tar.gz
;    http://ftp.debian.org/debian/dists/stretch/main/installer-i386/current/images/netboot/netboot.tar.gz
;    http://ftp.debian.org/debian/dists/jessie/main/installer-i386/current/images/netboot/netboot.tar.gz
;    http://ftp.debian.org/debian/dists/wheezy/main/installer-i386/current/images/netboot/netboot.tar.gz
;-Require:
;  Internet access
;-Notes:
; -
[PXESERVA_MENU_ENTRY]
asset    = Debian 12 Netboot
platform = amd64
;platform = i386

kernel_bios    = /NWA_PXE/$HEAD_DIR$/debian-installer/amd64/linux
append_bios    = initrd=/NWA_PXE/$HEAD_DIR$/debian-installer/amd64/initrd.gz priority=low vga=788
;kernel_bios    = /NWA_PXE/$HEAD_DIR$/debian-installer/i386/linux
;append_bios    = initrd=/NWA_PXE/$HEAD_DIR$/debian-installer/i386/initrd.gz priority=low vga=788

kernel_efi64   = /NWA_PXE/$HEAD_DIR$/debian-installer/amd64/linux
append_efi64   = initrd=/NWA_PXE/$HEAD_DIR$/debian-installer/amd64/initrd.gz priority=low vga=788
;kernel_efi32   = /NWA_PXE/$HEAD_DIR$/debian-installer/i386/linux
;append_efi32   = initrd=/NWA_PXE/$HEAD_DIR$/debian-installer/i386/initrd.gz priority=low vga=788
```

**Important**. Change the properties of the folder to enable sharing them. Under advance setting. 


<div align="center">

<img src="images/sharing-folder.png" alt="Windows Folder Sharing Settings" width="400"/><br>
<sub><b>Figure 5:</b> Windows folder sharing configuration for SERVA_ROOT</sub>

</div>


## Step 5: start the VM

<div align="center">

<img src="images/serva-install.png" alt="Debian Installer via Serva PXE Boot" width="500"/><br>
<sub><b>Figure 6:</b> Debian installer running on VM via Serva PXE boot</sub>

</div>

## Step 6: Complete the Debian Installation

### Installation Progress
- **Phase 1:** "Checking the Debian archive mirror" - ✅ Completed
- **Phase 2:** "Downloading Release files" - ✅ In Progress
- **Next Steps:** Standard Debian installation workflow

## Galería de Instalación (Imágenes en miniatura)

Cuando hice la configuración puse el GNOME sin querer y ya no pude quitarlo. 

<div align="center">

<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_12_32_55.png" alt="Paso 1" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_12_07.png" alt="Paso 2" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_12_19.png" alt="Paso 3" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_12_37.png" alt="Paso 4" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_12_44.png" alt="Paso 5" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_13_26.png" alt="Paso 6" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_13_43.png" alt="Paso 7" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_14_21.png" alt="Paso 8" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_14_42.png" alt="Paso 9" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_14_51.png" alt="Paso 10" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_15_11.png" alt="Paso 11" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_15_19.png" alt="Paso 12" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_15_31.png" alt="Paso 13" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_15_55.png" alt="Paso 14" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_16_04.png" alt="Paso 15" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_16_21.png" alt="Paso 16" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_16_30.png" alt="Paso 17" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_16_39.png" alt="Paso 18" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_16_54.png" alt="Paso 19" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_22_43.png" alt="Paso 20" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_23_00.png" alt="Paso 21" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_23_48.png" alt="Paso 22" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_24_05.png" alt="Paso 23" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_24_12.png" alt="Paso 24" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_24_39.png" alt="Paso 25" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_25_00.png" alt="Paso 26" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_26_11.png" alt="Paso 27" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_32_06.png" alt="Paso 28" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_33_14.png" alt="Paso 29" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_33_46.png" alt="Paso 30" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_34_03.png" alt="Paso 31" width="200"/>
<img src="images/debian/VirtualBox_DebianPXE_21_08_2025_14_45_28.png" alt="Paso 32" width="200"/>

</div>

## Step 7: 
## Preguntas

1. ¿Por que se tuvo que desactivar el DHCP del router principal?
En este caso se hizo con Apache y también usamos un DHCP ¿Que opciones existen que
pueda implementarse con TFTP?

En mi caso tuve que utilizar DHCP para que pudiera funcionar con Windows Serva.La configuración de Serva fue lo suficientemente específica y rápida para manejar las peticiones PXE sin necesidad de desactivar el DHCP del router principal. 

2. ¿Que es el fichero /srv/ ? ¿Qu ́e otros servicios se alojan ah ́ı?
En este caso se uso un netboot para que booteara bien el sistema y ocupamos internet para que pudiera descargas los paquetes ¿Hay forma de instalar el sistema operativo de una ISOn sin internet? Si es asi ¿Como sería?

Si de hecho se puede usar una red local e instalar el iso en una carpeta. La documentación para hacerlo con Serva esta en su página principal. 

3. ¿Cual es la diferencia ente TFTP, FTP y SFTP? Describa cada una.

TFTP. *Protocolo trivial de transferencia de archivos*. Un protocolo más simple y liviano utilizado principalmente para transferir archivos en una red local.

FTP. *Protocolo de transferencia de archivos*. Un protocolo de red estándar utilizado para transferir archivos de un host a otro a través de una red basada en TCP.

SFTP. *Protocolo de transferencia de archivos SSH*. Es una version segura de trasferencia que utiliza FTP. 

La diferencia central es la seguridad y la complejidad de las tareas que queramos trasferir. 

### File Dependencies
- **pxelinux.0:** Initial PXE bootloader
- **linux:** Debian kernel for installation
- **initrd.gz:** Initial filesystem with drivers and utilities
- **ServaAsset.inf:** Serva configuration for automatic discovery


## References
- [Serva PXE Documentation](https://www.vercot.com/~serva/an/NonWindowsPXE3.html)
- [Debian Netboot Installation Guide](https://www.debian.org/distrib/netinst)
- [PXE Boot Protocol Specification](https://tools.ietf.org/html/rfc4578)
- [Setting up TFTP to network install Ubuntu Mate using Serva on Windows](https://www.youtube.com/watch?v=fRV-LbHS6go)


---
**Project Status:** ✅ Successfully Completed  
**Implementation Date:** August 21, 2025  
**Verification:** Debian installer running and downloading packages