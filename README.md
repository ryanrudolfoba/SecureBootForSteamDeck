# How to Enable Secure Boot For Steam Deck

## About
This repository contains the instructions on how to generate the Platform Key (PK), Key Exchange Key (KEK) and Signature Database (DB) to enable the Secure Boot functionality in Steam Deck.

By default, the Steam Deck does not contain the keys needed for SecureBoot. It is missing the PK, KEK and DB keys. Without these keys, the SecureBoot functionality cannot be enabled.

## Disclaimer
Do this at your own risk!

## Requirements
1. USB flash drive. This will contain the installer / Linux ISO.
2. Another USB flash drive. This will be the target where Linux will be installed.
2. USB C dock / hub, mouse and keyboard.
3. sbctl to generate and install the needed keys.
4. some basic knowledge on how to install a Linux distro (Fedora / Ubuntu etc etc).

## Instructions - preparing and installing the USB flash drive for Linux
1. Download your favorite Linux distro and save it to your desktop / laptop. (I used Fedora on my environment)
2. Use ventoy or rufus to burn the ISO to a USB flash drive. (I already have a microsd that is formatted for ventoy, so I just copy / paste the Fedora Linux ISO)
3. Connect everything to the USB C hub / dock - Steam Deck, keyboard, mouse, flash drive that contains the Linux ISO, flash drive that will be the target for the Linux install
4. While the Steam Deck is powered off, press VOL+ and POWER button at the same time to access the boot device menu.
5. Use the DPAD / keyboard to select the flash drive that contains the Linux ISO and press ENTER.
6. Wait for the Linux ISO to load and then install Linux to the target flash drive.
7. Shutdown the Steam Deck once Linux is installed to the other flash drive.
8. Disconnect the flash drive that contains the Linux ISO as we dont need this anymore.
9. While the Steam Deck is powered off, press VOL+ and POWER button at the same time to access the boot device menu.
10. Use the DPAD / keyboard to select the flash drive where Linux is installed and press ENTER.

## Instructions - compiling sbctl (this instructions are for Fedora, it will be similar for other Linux distro)
1. Open terminal.

2. Install needed dependencies for sbctl.

    sudo dnf install git asciidoc go util-linux binutils
![image](https://user-images.githubusercontent.com/98122529/207431876-6429186b-3b14-4eea-98ba-657b8dcc8bb7.png)


3. Clone the sbctl repository.

    git clone https://github.com/Foxboron/sbctl.git
![image](https://user-images.githubusercontent.com/98122529/207431958-9707afb0-40e4-4452-bfef-0faa02bfb3d4.png)


4. Compile and install the sbctl package. This will take a few minutes.

    cd sbctl
    
    sudo make install
    
    ![image](https://user-images.githubusercontent.com/98122529/207433293-cc8534e6-3f68-413d-9933-6213f7496592.png)



## Instructions - using sbctl to generate and enroll keys
1. Open terminal.

2. Check the sbctl status. (Mine will show a little bit different as I have already installed sbctl and have generated keys)

    sbctl status
    
    ![image](https://user-images.githubusercontent.com/98122529/207450109-7f22ea6a-acbd-45ea-9b54-0cfde22ddb45.png)
    
    What does this mean?
    
    Installed - this will show installed if sbctl has already generated keys in /usr/share/secureboot
    
    Owner GUID - random GUID for key signing and setting up the owner of the keys
    
    Setup Mode - default is Enabled. Enabled means Secure Boot is not active and it is waiting for keys to be installed.
    
    Secure Boot - default is Disabled. Disabled means there are no keys active and Secure Boot is set to disabled state. 

3. Generate keys

    sbctl create-keys

    ![image](https://user-images.githubusercontent.com/98122529/207436579-4970f8ee-2e6b-4112-aa40-3e5147417462.png)


4. Enroll the keys
    
    sudo chattr -i /sys/firmware/efi/efivars/{PK,KEK,db}*
    
    sbctl enroll-keys -m

    ![image](https://user-images.githubusercontent.com/98122529/207435875-c0df839c-7dcf-483c-beae-e43074b5b45a.png)

5. Check again the sbctl status. This will now show a different status

    sbctl status

    ![image](https://user-images.githubusercontent.com/98122529/207436134-60403f50-bea1-454b-943c-19cc7d9dfa2f.png)

    What does this mean?
    
    Installed - this will show installed if sbctl has already generated keys in /usr/share/secureboot
    
    Owner GUID - random GUID for key signing and setting up the owner of the keys
    
    Setup Mode - this now shows DISABLED. It means that the keys needed for Secure Boot has been installed and active.
    
    Secure Boot - this will still show DISABLED. Reboot for the changes to take effect.
    
    Vendory keys - this now shows we are using microsoft keys.
    

## Instructions - use mokutil to query the keys
1. Open terminal.

2. Query the PK, KEK and db to make sure it is installed and available
    
    mokutil --pk
    
    ![image](https://user-images.githubusercontent.com/98122529/207453256-85d01ae2-d1d7-44b5-a015-d1ad1ffe728c.png)

    mokutil --kek
    
    ![image](https://user-images.githubusercontent.com/98122529/207453441-80c6f0f9-235a-4bd4-9b16-f756bf3b991c.png)

    mokutil --db
    
    ![image](https://user-images.githubusercontent.com/98122529/207453515-897e1a95-c8d8-483d-84d3-a876796d82ee.png)

3. If everything looks good then the next step is to sign the EFI loader and kernel or else you won't be able to boot any OS!


## Instructions - using sbctl to sign the EFI loader and kernel
THIS STEP IS VERY IMPORTANT!!! If you don't sign your EFI loader and kernel then you won't be able to boot to Windows or Linux!!!

1. Open terminal.

2. Query sbctl the status of EFI entries. This will show not signed (but for me I've already signed several EFI entries)

    sudo sbctl verify
    
    ![image](https://user-images.githubusercontent.com/98122529/207457975-cea9f50c-6f64-4504-a295-f85197cbe840.png)

    What does this mean?
    
    BOOTIA32.EFI this is for Windows (32bit? I signed it anyway just to be sure)
    BOOTX64.EFI this is for Windows
    vmlinuz-5.14.10-300.fc35.x86_64 this is the Linux kernel I am using
    
3. Sign the EFI and kernel using sbctl

    sudo sbctl sign -s /boot/efi/EFI/BOOT/BOOTX64.EFI
    
    sudo sbctl sign -s /boot/efi/EFI/BOOT/BOOTIA32.EFI
    
    sudo sbctl sign -s /boot//vmlinuz-5.14.10-300.fc35.x86_64
    
    ![image](https://user-images.githubusercontent.com/98122529/207455939-7ad82816-02a4-4727-93ab-e9ce9ebe7fac.png)

4. If no error, reboot to Windows and SecureBoot status will now show Enabled!
![image](https://user-images.githubusercontent.com/98122529/207457048-d9abd5ea-d2e9-415a-8d97-156c07645fff.png)


## Instructions - revert changes and disable Secure Boot
1. Open terminal.

2. Install efitools

    sudo dnf install efitools
    
    ![image](https://user-images.githubusercontent.com/98122529/207459225-088fa41c-927a-4292-89f4-4926ac7227d8.png)

3. Delete the PK, KEK and db

    sudo chattr -i /sys/firmware/efi/efivars/{PK,KEK,db}*
    
    sudo efi-updatevar -d 0 -k /usr/share/secureboot/keys/PK/PK.key PK
    
    sudo efi-updatevar -d 0 -k /usr/share/secureboot/keys/KEK/KEK.key KEK
    
    sudo efi-updatevar -d 0 -k /usr/share/secureboot/keys/db/db.key db (you might need to do this twice to clear the microsoft vendor keys)
    
    ![image](https://user-images.githubusercontent.com/98122529/207460049-8414edcd-297d-4745-abd5-440b66db28eb.png)

4. Check the sbctl status. This is now back to the default settings.

    sbctl status
    
    ![image](https://user-images.githubusercontent.com/98122529/207460220-461ec1a1-6d5b-4ef9-b10c-4b7484eb3083.png)

5. Use mokutil to query the PK, KEK and DB make sure they don't exist.

    ![image](https://user-images.githubusercontent.com/98122529/207460372-c70c1e9b-50ef-4f1f-9fa2-ef5da0bd7bf5.png)

6. Reboot and Secure Boot will be disabled and back to factory defaults - no PK, KEK and db keys.
    

    
