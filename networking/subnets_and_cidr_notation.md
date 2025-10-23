# Subnets and CIDR Notation

Just wanted to make sure I have a solid understanding of this because I forget from time to time how the theory works.

CIDR notation looks something like this:

```text
192.168.1.0/24
```

- the `/24` portion is the CIDR notation.
- from the above you should be able to determine that
  - the **network** is `192.168.1`
  - the **host** is `.0`

What does this notation mean? It's basically short-hand for saying, this range of ip-addresses:

```
192.168.1.0
192.168.1.1
...
192.168.1.254
192.168.1.255 <- this is actually reserved for the broadcast address
```

How this works:

```text
ip address                              -> 192.168.1.0
ip address in binary form               -> 11000000.10101000.00000001.00000000
/24 means the first 24 bits are 1's     -> 11111111.11111111.11111111.00000000
```

The subnet masks essential tells us that the the first 24 bits that line up with the ones is the network:

```text
ip address      -> 11000000.10101000.00000001.00000000
subnet mask /24 -> 11111111.11111111.11111111.00000000
                   -----------------------------------
                   [       network           ][ host  ]


# in binary form:
network ip address -> 11000000.10101000.00000001
hosts ip address   -> .00000000

# in decimal form:
network ip address -> 192.168.1.0
hosts ip address   -> .0
```

Quick formulas:
- Total addresses = 2^(32 - prefix). Example: /24 → 2^(32-24) = 256.
- Usable hosts = Total addresses - 2 (network + broadcast). Example: /24 → 254 usable.
- Exceptions: /31 (RFC 3021) is used for point-to-point links and /32 is a single-host route.

Example (calculator):
You can confirm with a subnet calculator (example set to /24):
https://www.calculator.net/ip-subnet-calculator.html?cclass=any&csubnet=24&cip=192.168.1.0&ctype=ipv4&x=Calculate

Typical /24 output:
```
IP Address:	192.168.1.0
Network Address:	192.168.1.0
Usable Host IP Range:	192.168.1.1 - 192.168.1.254
Broadcast Address:	192.168.1.255
Total Number of Hosts:	256
Number of Usable Hosts:	254
Subnet Mask:	255.255.255.0
Wildcard Mask:	0.0.0.255
Binary Subnet Mask:	11111111.11111111.11111111.00000000
IP Class:	C
CIDR Notation:	/24
IP Type:	Private

Short:	192.168.1.0 /24
Binary ID:	11000000101010000000000100000000
Integer ID:	3232235776
Hex ID:	0xc0a80100
in-addr.arpa:	0.1.168.192.in-addr.arpa
IPv4 Mapped Address:	::ffff:c0a8.0100
6to4 Prefix:	2002:c0a8.0100::/48
```




## References

- https://www.cbtnuggets.com/blog/technology/networking/proper-cidr
- https://www.calculator.net/ip-subnet-calculator.html
- https://www.geeksforgeeks.org/computer-networks/role-of-subnet-mask/
