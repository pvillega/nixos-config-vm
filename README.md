# NIXOS Configuration

This document describes how to install a Scala dev environment in [VirtualBox](https://www.virtualbox.org). This is aimed ot environments in which the company locks down the laptop a bit too much, and the only solution is to run your own OS with full rights in a vm.

You can check the [NixOS Manual](https://nixos.org/nixos/manual/) if you are stuck.

## Requirements

Download [NixOS](https://nixos.org/nixos/download.html). Version used in this document is 17.09, Graphical installation CD.

Install [VirtualBox](https://www.virtualbox.org) (version tested for this document is 5.2.2). Other tools may work (like [Parallels](https://www.parallels.com/uk/)) but you are on your own ;)

## Set up your VM

Create a new VM with the following settings:

* Linux 2.6 64 bits
* 12000 Mb RAM assigned (for a laptop with 16 Gb this is the maximum allowed by VBox without any complains)
* VDI Hard drive, dynamically allocated, at least 150 Gb (we assume a host machine with at least 200 Gb hard drive)

Once created, click on Settings and on System / Processor:

* assign as many CPU as the VM allows you (usually 4)
* enable PAE/NX

## Installing NixOS on the VDI

Load the NixOS iso image on your virtual box instance and start it to install NixOS. The installer will quickly set up things and open a terminal prompt as *root*

Start the GUI with: `systemctl start display-manager`

Open `GParted` and:

* create a new partition table (Device / New Partition Table) of type `msdos`
* add a new partition of type `linux-swap` for the swap area. Label it `swap`. 4 Gb should be enough.
* add a new partition of type `ext4` to the virtual drive with label `nixos`
* apply changes

Enable swap on the labelled `swap` partition by running:
`# swapon /dev/disk/by-label/swap

Mount the partition by the label assigned (if you forgot to label it, you can do so now with GParted)

`# mount /dev/disk/by-label/nixos /mnt`

Generate default config:
`# nixos-generate-config --root /mnt`

Edit the generated config: 
`# nano /mnt/etc/nixos/configuration.nix`

and uncomment the device to use on boot:

```
boot.loader.grub.device = "/dev/sda";
```

Then start the installation:
`# nixos-install`

The last step allows you to set up a password for root (on our vm: Bollocks)

Once done, eject the ISO from VirtualBox and reboot the machine.


## Create your own account

Up to now you have done all the above steps as `root`. To create your own account use:

```
# useradd -c 'Name Surname' -m accountId
# passwd accountId
```

## Core packages and VirtualBox tools

Run these commands to add new sources for packages:

```
nix-channel --add https://nixos.org/channels/nixos-17.09 nixos
nix-channel update
nixos-rebuild switch
```

Install VirtualBox Guest Addition with `nix-env -iA nixos.linuxPackages.virtualboxGuestAdditions`

Then to automatically mount folders from your host machine:

* add a new virtual box shared folder (under Settings / Shared Folders in virtual box)
* edit `/etc/nixos/configuration.nix` and add before the last '}'

```
  fileSystems."/virtualboxshare" = {
    fsType = "vboxsf";
    device = "nameofthesharedfolder";
    options = [ "rw" ];
  }; 
```

## Copy our configuration

The required configuration for NixOS is provided alongside this documentation in the repository.

Edit configuration file at `/etc/nixos/configuration.nix` and copy the configuration (see companion file to this document).

Then run `nixos-rebuild switch` as root to make that configuration your current one. You can now shutdown the vm, unload the iso image and start the vm again. You should see a working NixOS instance.


## Relevant commands

You can check the [NixOS Manual](https://nixos.org/nixos/manual/) for reference on common tasks.

When modifying your configuration at */etc/nixos/configuration.nix* :

* `nixos-rebuild build` compiles the configuration but doesn't load any changes. To verify syntax is correct.
* `nixos-rebuild test` rebuilds and loads the configuration but doesn't make it *final*, changes will be lost on reboot.
* `nixos-rebuild switch` loads the new configuration and makes it the default one
* `nixos-rebuild switch --upgrade` loads the new configuration, upgrades packages, and makes it the default one

You can set the source channel for package sfor your NixOS with `nixos-channel --add https://nixos.org/channels/nixos-17.09 nixos`. When you do a `nixos-rebuild switch --upgrade` it will use the channel to upgrade your packages accordingly. This is effectively the way to upgrade to a newer NixOS version: use a new channel with a newer version, request an upgrade.

You can list available packages via `nix-env -qaP '*' --description`. 
To filter packages use `nix-env -qaP '.*Expression.*' --description`. Note the '.' before the '*'.
You can also use the website [https://nixos.org/nixos/packages.html](https://nixos.org/nixos/packages.html) for live search. It provides installation instructions for packages.

You can start shells with specific packages loaded only for that shell session:
`nix-shell -p python27Packages.numpy`
