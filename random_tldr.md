VPN = Virtual Private Network
IKE = Internet Key Exchange (v1/v2)
  => protocol used to set up a security association (SA) in the IPsec protocol suite.
  => protocol that allows for direct IPsec tunneling between the server and client.


IPsec = Internet Protocol Security (IPsec)
  => a network protocol suite that authenticates and encrypts the packets of data sent over a network.

StrongSwan => an open source IPsec daemon

----

## Setup an IKEv2 VPN Server with StrongSwan on Ubuntu 16.04

Source: https://www.digitalocean.com/community/tutorials/how-to-set-up-an-ikev2-vpn-server-with-strongswan-on-ubuntu-16-04

----

## Setup initial server with Ubuntu

Source: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04

- Login as `root`

    ssh root@your_server_ip

  Or after you logged in to your server you can run

    $ sudo su

- Create new user
  As `root`, run command below (replace the bracket your value)
  
    # adduser [newuser]

  You will be prompted to setup password and other additional info about the user

- Setup root privileges
  Add new created user to `sudo` group (still as `root`)

    # usermod -aG sudo [newuser]

- Adding public key authentication (Optional but recommended)

  Basically, this step only ask us to register client SSH public key to our server
  inside `~/.ssh/authorized_keys` file.

  Stil as `root`, we will switch to newly created user and setup `ssh` directory

    # sudo - [newuser]
    $ mkdir ~/.ssh
    $ chmod 700 ~/.ssh
    $ touch ~/.ssh/authorized_keys
    $ chmod 600 ~/.ssh/authorized_keys
    $ nano ~/.ssh/authorized_keys

  Then paste client SSH public key into file above
  
- Disable Password Authentication (Recommended)

  As `root` or newly created user from step above
    
    $ sudo nano /etc/ssh/sshd_config

  and edit config like below

    PasswordAuthentication no
    PubkeyAuthentication yes
    ChallengeResponseAuthentication no

  then reload SSH daemon
    
    $ sudo systemctl reload sshd

- Test SSH login

  Test login into server with newly created user via SSH

    ssh [newuser]@[your_server_ip]
  
- Setup a basic Firewall (UFW)

  Check available application
    
    $ sudo ufw app list

  Allow application listed from command above
    
    $ sudo ufw allow OpenSSH

  Enable the firewall, then press "y" if prompted
    
    $ sudo ufw enable

  Check firewall status. Add `-v` to make the output more detail

    $ sudo ufw status

----

## `iptables`

sources: https://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/

Types of chains:

- Input
  This chain is used to control behaviour for incoming connections. For example, if a user attempts to SSH into you server/PC,
  iptables will attempt to match the IP address and port to a rule in the input chain.

- Forward
  This chain is used for incoming connections that aren't actually being delivered locally.
  Think of a router, data is always is being sent to it but rarely actually destined for the router itself, the data just
  forwarded to its target.

  You will use this chain if you're doing some kind of NATing or something else on your system that requires forwarding.
  You can check the needs this chain with `iptables -L -v`
  
- Output
  This chain is used for outgoing connections.
  e.g you want to access google.com, iptables will check its output chain to see what the rules are regarding ping and
  google.com before making a decision to allow or deny the connection attempt.

Caveats:
You need to setup output and input chain properly.

Default commands to accept connections:
```
$ iptables --policy INPUT ACCEPT
$ iptables --policy FORWARD ACCEPT
$ iptables --policy OUTPUT ACCEPT
```

There are the three most basic and commonly used "responses":

- Accept
  Allow the connection

- Drop
  Drop the connection, act like it never happened. This is the best if you don't want the source to realize your system exists.

- Reject
  Don't allow the connection, but send back an error. This is the best if you don't want a particular source to connect to your system,
  but you want them to know your firewall blocked them.

### iptables cheatsheet:

#### Block all connections from specific ip address
```
iptables -A INPUT -s 10.2.2.2 -j DROP
```

#### Block all connections from range of ip addresses
```
iptables -A INPUT -s 10.2.2.2/24 -j DROP

# or

iptables -A INPUT -s 10.2.2.2/255.255.255.0 -j DROP
```

#### Block SSH connections from specific ip address
```
iptables -A INPUT -p tcp --dport ssh -s 10.2.2.2 -d DROP
```

You can replace `ssh` with any protocol or port number.
The `tcp` part tells iptable what kind of connection the protocol use. If you were blocking a protocol
rather that uses UDP rather than TCP, then `-p udp` would be necessary instead.

#### Block SSH connections from any ip address
```
iptables -A INPUT -p tcp --dport ssh -j DROP
```

#### List current config iptables 
```
iptables -L -v -n
```
`-v` option will give you packet and byte information.
`-n` option will list everything numerically.

-----

## Stateful and Stateless app

Stateful app => app that saves client data from activities of one session for use in the next session.
The data is saved is called *apps state*.

Stateless app => the opposite of stateful app, this app doesn't record any previous state

----

## DNS BIND (Berkeley Internet Name Domain)

BIND itself is an open source software that enables you to publish your Domain Name System (DNS)
information on the internet, and to resolve DNS queries for your users.

BIND has three parts:
- Domain Name Resolver: resolves questions about names by sending those questions to appropriate servers
and responding appropriately to the servers replies.
- Domain Name Authority server: an authoritive DNS server answers request from resolvers, using information
about the domain names it is authoritive for.
- Tools


----

## DNS (Domain Name System)

On simple terms, DNS is used to translate IP address which is hard to read and remember by human into something
easier for human to read and remember, such as google.com and others.

DNS servers => where your computer search IP address for specific DNS (usally provided by ISP (Internet Service Providers) or the router could be the DNS server but still forward it to ISP)

Computers cache DNS locally

IANA (Internet Assigned Numbers Authority) => organization under the IAB (Internet Architecture Board) of the Internet Society that, under a contract from the U.S government, has overseen the allocation of Internet Protocol addresses to ISP. IANA also has had responsibility for the registry for any "unique parameters and protocol values" for internet operation. These including port numbers, character sets, and MIME media access type.

#### DNS Records

DNS records => a database record used to map a URL to an IP address and this is stored on DNS servers.

Most common record types:
- A (address) => used to map a hostname to an IP address. Generally, A record are IP addresses.
  If a computer consists of multiple IP addresses, adapter cards or both, it must possess multiple address records.

- CNAME (canonical name) => can be used to set an alias for the hostname.

- MX (mail exchanges) => permits mail to be sent to the right mail servers located in the domain.
  Other than IP addresses, MX records include fully-qualified domain names.

- NS (name server) => describea a name server for the domain that permits DNS lookups within several zones.
  Every primary as well as secondary name server must be reported via this record.

- PTR (pointer) => creates a pointer, which maps an IP address to the host name in order to do reverse lookup.

- SOA (start of pointer) => declares the most authoritative host for the zone. Every zone file should include an SOA record.

- TXT (text record) => permits the insertion of arbitrary text into a DNS record. These records add SPF records into a domain.


Other record types:
- TTL (time-to-live) => sets the period of data, which is ideal when a recursive DNS server queries the domain name information.

#### DNS Zones

DNS zone => contains all the domain names the domain with the same domain name contains, except for domain names in delegated subdomains.

ex: 
The top level domain ca (for Canada) has subdomains called ab.ca, on.ca, and qc.ca.
Authority for the ab.ca, on.ca and qc.ca domains may be delegated to nameservers in each province.
The domain ca contains all the data in ca plus all the data  in ab.ca, on.ca and qc.ca.
However, the zone ca contains only the data in ca, which is probably mostly pointers to the delegated subdomains ab.ca, on.ca and qc.ca are separate zones from the ca zone.

** Don't confuse DNS Zone with DNS Domains **
DNS zone can contain multiple domains or just one domain, the important thing to remember is that it is used for delegating control of portions of the namespace.
DNS zone are used to delegate administrative rights to different parts of the namespace.

#### DNS Forward & DNS Reverse

DNS forward => type of DNS request in which a domain name is used to obtain its corresponding IP address.
A forward DNS request is the opposite of a reverse DNS lookup.

DNS reverse => using an IP address to find a domain name.


