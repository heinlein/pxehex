# Create IPv4 hex filenames suitable for PXELINUX

The PXE network booting system distributed by [the Syslinux Project](http://syslinux.zytor.com/wiki/index.php/The_Syslinux_Project) is widely used for installing operating systems on networks of all sizes. It's used in conjunction with DHCP and TFTP servers.

The DHCP server will tell the client system where to look for the TFTP server from which boot files can be loaded. First, the client's PXE stub will download the bootloader image (typically named `pxelinux.0` or `gpxelinux.0`) from the TFTP server. Once the client executes the bootloader, it will then search for its configuration file.

The PXELINUX bootloader will look for a [succession of files](http://www.syslinux.org/wiki/index.php?title=PXELINUX#Configuration). You should read the official documentation to get the full story, but the short version is that their filenames are based on

1. Client UUID (not always present)
1. Client Ethernet MAC (prefixed by ARP type, typically `01`)
1. IPv4 address in uppercase hexidecimal (on a widening scope)
1. A file named 'default'

The part that usually trips me up is converting an IPv4 address to hex. So I hacked up the script in this repository to assist me.

## Usage

Invoking the script to do useful work requires a fully qualified domain name or a full IPv4 address:

```nohighlight
usage:
  pxehex [-h]
  pxehex [-a] [-v] FQDN
  pxehex [-a] [-v] IPv4 address
```

When passed a FQDN or IPv4 address with no other arguments, the script will return an 8-character hexadecimal string that PXELINUX will associate with that address.

```nohighlight
[bash ~]$ pxehex 192.168.25.101
C0A81965
```

With the `-v` option, you'll also get a comment the IP address in question.

```nohighlight
[bash ~]$ pxehex -v madboa.com
6883B274 # 104.131.178.116/32
```

Finally, the `-a` option will produce all of the address-based filenames for which PXELINUX will search for your address/hostname. Adding `-v` will clarify the scope for each filename.

```nohighlight
[bash ~]$ pxehex -a -v 10.100.200.55
0A64C837 # 10.100.200.55/32
0A64C83  # 10.100.200.48/28
0A64C8   # 10.100.200.0/24
0A64C    # 10.100.192.0/20
0A64     # 10.100.0.0/16
0A6      # 10.96.0.0/12
0A       # 10.0.0.0/8
0        # 0.0.0.0/4
```

## Caveats

I've only tested the script using `bash`. It may fail when run using other shells. It may rely on other command-line utilities, depending on the options you invoke:

* `dig` is required if you pass a hostname to the script;
* `seq` is required to use the `-a` option;
* `ipcalc` is required to use the `-a` and `-v` options together.

The script currently works only with the `ipcalc` found in the Red Hat `initscripts` package. A future version will work with the version installed on Ubuntu systems or via MacPorts.

<!-- vim: set filetype=markdown: -->

