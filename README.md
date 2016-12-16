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


```sh
#!/bin/bash
#
# returns the PXE-friendly hex version of a valid IPv4 address or
# a fully qualified domain name that can be resolved via its A
# record in DNS.
#
# note: requires the 'dig' and 'ipcalc' utilities for full functionality
#       but there are a couple different ipcalc versions out there. we
#       rely on the one used by red hat in the initscripts rpm.
#
# ======================================================================

# name of script
ME=$(basename $0)
# we use dig when resolving hostnames
DIG="dig"
# use ipcalc (from initscripts rpm) for network calculations
IPCALC="ipcalc"
# flag for printing all possible hex versions
SHOWALL=0
# flag for adding comments output
COMMENTS=0


# give a brief usage message if user passes no argument
function usage {
  local RETVAL=0
  if test "$1" -gt 0; then
    RETVAL=$1
  fi
  echo "usage:"
  echo "  $ME [-h]"
  echo "  $ME [-a] [-v] FQDN"
  echo "  $ME [-a] [-v] IPv4 address"
  echo ""
  echo "returns PXE-friendly hex version of IP address"
  echo ""
  echo "-a  returns all possible hex-based filenames for target address"
  echo "-v  includes comments about address range suitable for filename"
  echo ""
  echo "PXELINUX will search for a list of filenames beginning with the"
  echo "specific address and widening in scope by subtracting 4 fom the"
  echo "current CIDR block: /32, then /28, /24, etc."
  exit $RETVAL
}

while getopts ":hva" KEY; do
  case "$KEY" in
    h)
      usage
      ;;
    a)
      SHOWALL=1
      ;;
    v)
      COMMENTS=1
      ;;
    *)
      echo "ERROR: Unknown option: -$OPTARG" 1>&2
      exit 1
      ;;
  esac
done

if test $SHOWALL -gt 0 -a $COMMENTS -gt 0; then
  which $IPCALC >/dev/null 2>&1
  if test $? -ne 0; then
    echo "$ME requires the '$IPCALC' utility." 2>&1
    echo "Please install it or adjust your PATH variable accordingly." 2>&1
    exit 1
  fi
fi 

shift $((OPTIND-1))

# our target address or hostname
TGT=$1
# give a brief usage message if user passes no argument
if test -z "$TGT"; then
  usage 1
fi

# any script argument containing an alphabetic character will be
# treated as a hostname (which can be resolved using dig); arguments
# without letters are assumed to be IPv4 addresses.
ALPHA=$(expr "$TGT" : '.*[[:alpha:]]')
if test $ALPHA -gt 0; then
  which $DIG >/dev/null 2>&1
  if test $? -ne 0; then
    echo "$ME requires the '$DIG' utility." 2>&1
    echo "Please install it or adjust your PATH variable accordingly." 2>&1
    exit 1
  fi
  IP=$($DIG $TGT -t A +short)
else
  IP=$TGT
fi

# a valid IPv4 addr will contain exactly three dots
DOTS=${IP//[^.]}
NUMDOTS=${#DOTS}
if test $NUMDOTS -ne 3; then
  echo "Invalid hostname or IPv4 address. Exiting now." 2>&1
  exit 1
fi

# turn IPv4 address into a space-separated set of numbers suitable
# for looping in bash
NUMLIST=$(echo $IP | sed 's/\./ /g')

# use printf to get zero-padded hex value for each of the four
# numbers in the IP address. tack each of those values onto
# the HEX variable, which is ultimately what this script aims
# to produce
HEX=""
for N in $NUMLIST; do
  NEWHEX=$(printf '%02x' $N | tr a-f A-F)
  HEX=${HEX}${NEWHEX}
done

# the length of HEX should be 8, but we'll let bash figure
# it out anyway
HEXLEN=${#HEX}

if test $SHOWALL -gt 0; then
  # drop one character from the hex filename at a time,
  # recalculating the netmask and base network each time if
  # the -v option was invoked.
  for N in $(seq $HEXLEN -1 1); do
    printf "%-${HEXLEN}s" ${HEX:0:$N}
    if test $COMMENTS -gt 0; then
      MASK=$(expr $N \* 4)
      NW=$($IPCALC -s -n ${IP}/${MASK} | cut -d= -f2)
      printf " # %s/%s" $NW $MASK
    fi
    echo
  done
else
  printf "%-${HEXLEN}s" $HEX
  if test $COMMENTS -gt 0; then
    printf " # %s/%s" $IP 32
  fi
  echo
fi

### eof
```

<!-- vim: set filetype=markdown: -->

