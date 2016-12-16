# Create IPv4 hex filenames suitable for PXELINUX

The PXE network booting system distributed by [the Syslinux Project](http://syslinux.zytor.com/wiki/index.php/The_Syslinux_Project) is widely used for installing operating systems on networks of all sizes. It's used in conjunction with DHCP and TFTP servers.

The DHCP server will tell the client system where to look for the TFTP server from which boot files can be loaded. First, the client's PXE stub will download the bootloader image (typically named `pxelinux.0` or `gpxelinux.0`) from the TFTP server. Once the client executes the bootloader, it will then search for its configuration file.

The PXELINUX bootloader will look for a [succession of files](http://www.syslinux.org/wiki/index.php?title=PXELINUX#Configuration). You should read the official documentation to get the full story, but the short version is that their filenames are based on

1. Client UUID (not always present)
1. Client Ethernet MAC (prefixed by ARP type, typically `01`)
1. IPv4 address in uppercase hexidecimal (on a widening scope)
1. A file named 'default'

The part that usually trips me up is converting an IPv4 address to hex. So I hacked up the script in this repository to assist me.

## Caveats

The script may on other command-line utilities, depending on the options you invoke:

* `dig` is required if you pass a hostname to the script;
* `seq` is required to use the `-a` option;
* `ipcalc` is required to use the `-a` and `-v` options together.

Script currently works only with the `ipcalc` found in the Red Hat `initscripts` package. A future version will work with the version installed on Ubuntu systems or via MacPorts.

<!-- vim: set filetype=markdown: -->

