To compile and boot a custom Linux kernel in NixOS using the source you've specified, follow these steps:

**1. Prepare Your Environment**

Ensure your NixOS system is up-to-date:

```bash
sudo nix-channel --update
sudo nixos-rebuild switch
```

**2. Create a Custom Kernel Configuration File**

If you have a specific `.config` file for your kernel, ensure it's accessible on your system. If not, you can generate one:

- Start by creating a shell environment with the necessary tools:

  ```nix
  { pkgs ? import <nixpkgs> {} }:

  pkgs.mkShell {
    nativeBuildInputs = [ pkgs.ncurses pkgs.pkg-config ];
  }
  ```

- Enter the shell:

  ```bash
  nix-shell
  ```

- Download and extract the kernel source:

  ```bash
  wget https://github.com/Jackler87/linux/releases/download/v51/kernel-files-6.13.0-rc5-g28753b266944.tar.gz
  tar -xzf kernel-files-6.13.0-rc5-g28753b266944.tar.gz
  cd linux-6.13.0-rc5
  ```

- Generate the configuration file:

  ```bash
  make menuconfig
  ```

- After configuring, save the `.config` file to a known location, e.g., `/home/user/custom-kernel-config`.

**3. Define the Custom Kernel in NixOS**

Create a Nix expression to define your custom kernel. Save the following content to a file, e.g., `custom-kernel.nix`:

```nix
{ pkgs, ... }:

let
  customKernel = pkgs.linuxManualConfig {
    version = "6.13.0-rc5";
    src = pkgs.fetchurl {
      url = "https://github.com/Jackler87/linux/releases/download/v51/kernel-files-6.13.0-rc5-g28753b266944.tar.gz";
      sha256 = "sha256-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"; # Replace with the actual SHA-256 hash
    };
    configfile = /home/user/custom-kernel-config; # Path to your custom .config file
  };
in
{
  boot.kernelPackages = pkgs.recurseIntoAttrs (pkgs.linuxPackagesFor customKernel);
}
```

**4. Calculate the SHA-256 Hash**

Nix requires the SHA-256 hash of the source tarball:

```bash
nix-prefetch-url https://github.com/Jackler87/linux/releases/download/v51/kernel-files-6.13.0-rc5-g28753b266944.tar.gz
```

Replace the `sha256` value in `custom-kernel.nix` with the output of this command.

**5. Integrate the Custom Kernel into Your NixOS Configuration**

In your `configuration.nix`, import the custom kernel configuration:

```nix
{ config, pkgs, ... }:

{
  imports = [
    ./custom-kernel.nix # Adjust the path accordingly
  ];

  # Other configuration options...
}
```

**6. Rebuild Your System**

Apply the changes by rebuilding your NixOS system:

```bash
sudo nixos-rebuild switch
```

**7. Reboot**

After the rebuild completes, reboot your system to boot into the new custom kernel:

```bash
sudo reboot
```

**8. Verify the New Kernel**

After rebooting, verify that the system is running the new kernel:

```bash
uname -r
```

The output should match the version of your custom kernel, e.g., `6.13.0-rc5`.

**References:**

- [NixOS Wiki: Linux Kernel](https://nixos.wiki/wiki/Linux_Kernel)
- [NixOS Discourse: Build Linux with Custom Kernel Config](https://discourse.nixos.org/t/build-linux-with-custom-kernel-config/3076)

*Ensure that your custom kernel configuration is compatible with your system hardware and requirements to maintain system stability.* 
