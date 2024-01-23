# NixOS Offline ISO Builder
This repository provides a flake template for creating offline ISO images of the
NixOS Calamares installer with Gnome. 

It patches over v0.3.14 of the [Calamares NixOS Extensions](https://github.com/NixOS/calamares-nixos-extensions/tree/calamares)
to disable checking for an online connection, and then will install with user
provided configuration files.

## Usage
- [Install](https://nixos.org/download#download-nix) Nix or NixOS to your online build system and [enable flake](https://nixos.wiki/wiki/Flakes) support
- Clone this repo to a directory where you wish to build your ISO
- Copy your configuration files to the ./nix-cfg directory
- You will need to have a hardware-configuration.nix template file in this directory with your configuraiton.nix file
- The hardware-configuration.nix will be over-written by the installation process,
but it is needed to build the configuration for the ISO

### Build Dependencies
You must include system build dependencies in one of two ways:
1. In your configuration.nix file by declaring - 
```
  system.includeBuildDependencies = true;
```
This will include all the system dependencies as part of the configuration and
will allow you to both install from the ISO image and re-build your system 
configuration while offline (provided you don't add any dependencies). For more
information on this option, see [Linus Heckemann's blog](https://linus.schreibt.jetzt/posts/include-build-dependencies.html)

2. In the ISO's flake.nix file by declaring -
```
            isoImage = {
              contents = [
                {
                    source = ./nix-cfg;
                    target = "/nix-cfg";
                }
              ];
              storeContents = [ 
                config.system.build.toplevel
              ];
              includeSystemBuildDependencies = true;
            }
```
If you do not include dependencies in your configuration.nix, then they must
be declared here so they are included in the nix store for the ISO. However,
they will not be installed to the system as part of the installation process.
This will result in a much smaller nix store on your target system, but you will
not be able to re-build its configuration offline.

### Free Disk Space
Ensure your nix store partition has enough free space to build the ISO.
The ISO will be much larger than normal (20+ Gb) depending on what dependencies are included.
I recommend having at least 3x the ISO size available, or approx. 100+ Gb free.

### Modules
The installation script will recursively copy configuration files from either
/iso/nix-cfg or /tmp/nix-cfg to /etc/nixos on the target system. This allows
you to use any number of files and sub-directories for your configuration.

### Dynamic Configuration
- The installer will look for user configuration files in /tmp/nix-cfg prior to
searching /iso/nix-cfg. This allows you to dynamically change the system configuration
after booting into the installation environment.
- If configuration changes add dependencies, then the install will fail because
they will be missing from the ISO's nix store

## Limitations
Flake configurations are not supported. I have not found a way to make a flake
based system configuration work completely offline. There are also [open issues]
(https://github.com/NixOS/nix/issues/8953) related to this problem. If anyone
has a solution, I would be very interested in seeing it.
