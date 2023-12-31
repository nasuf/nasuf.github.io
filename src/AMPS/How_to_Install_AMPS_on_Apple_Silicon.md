# How to install AMPS on Apple Silicon 

The AMPS installation binary is designed for AMD x86 architecture, which is incompatible with Apple's M-series Silicon built on ARM architecture. However, there are two workarounds to implement this:

- [UTM virtual machine ](https://docs.getutm.app/installation/macos/)
  - UTM can be utilized to emulate a complete x86 Linux distribution on an ARM architecture system (macOS), though the performance is notably suboptimal.
- [qemu-user-static](https://github.com/multiarch/qemu-user-static)
  - This tool can be employed to emulate a standalone software running environment with x86 architecture. 

Please follow steps below to install qemu-user-static.

## Install necessary packages

```console
kali@kali:~$ sudo apt update
kali@kali:~$
kali@kali:~$ sudo apt install -y qemu-user-static binfmt-support
kali@kali:~$
kali@kali:~$ sudo dpkg --add-architecture amd64
kali@kali:~$
kali@kali:~$ sudo apt update
kali@kali:~$
kali@kali:~$ sudo apt install libc6:amd64
kali@kali:~$
```

Please keep in mind that more libraries may need to be installed depending on what package is being ran. In some cases these libraries will be installed automatically with the package install, however in others some research must be done to learn what is missing.

## Running x86 code

```console
# Before qemu-user-static install
kali@kali:~$ sudo dpkg --add-architecture amd64
kali@kali:~$
kali@kali:~$ sudo apt install -y powershell
kali@kali:~$
kali@kali:~$ file /opt/microsoft/powershell/7/pwsh
/opt/microsoft/powershell/7/pwsh: ELF 64-bit LSB pie executable, x86-64, version 1 (GNU/Linux), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=9c3feab2531f770c71d023f031faf37758181701, stripped
kali@kali:~$
kali@kali:~$ pwsh
zsh: exec format error: pwsh
kali@kali:~$
# After qemu-user-static install
kali@kali:~$ sudo apt install -y qemu-user-static binfmt-support
kali@kali:~$
kali@kali:~$ pwsh
PowerShell 7.1.3
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.


┌──(kali㉿kali)-[/home/kali]
└─PS>
```

If there is a downloaded binary that is x86 that is not automatically being ran under `qemu-user-static`, you can invoke it with the following command:

```console
kali@kali:~$ qemu-x86_x64-static my_x86_code
```