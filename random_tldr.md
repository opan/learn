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

#### Forwarding SSH for LXC container
```
sudo iptables -t nat -A PREROUTING -p tcp -i enp0s8 --dport 9001 -j DNAT --to 240.15.0.179:22
```

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

FQDN (Fully Qualified Domain Name) => an absolute domain name.

#### DNS Records

DNS records => a database record used to map a URL to an IP address and this is stored on DNS servers.

Most common record types:
- A (address) => used to map a hostname to an IP address. Generally, A record are IP addresses.
  If a computer consists of multiple IP addresses, adapter cards or both, it must possess multiple address records.

- CNAME (canonical name) => can be used to set an alias for the hostname.

- MX (mail exchanges) => permits mail to be sent to the right mail servers located in the domain.
  Other than IP addresses, MX records include fully-qualified domain names.

- NS (name server) => describe a name server for the domain that permits DNS lookups within several zones.
  Every primary as well as secondary name server must be reported via this record.

- PTR (pointer) => creates a pointer, which maps an IP address to the host name in order to do reverse lookup.

- SOA (start of authority) => declares the most authoritative host for the zone. Every zone file should include an SOA record.

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

#### DNS Zone file structure and record contents

Directive begin with `$`. List directives:
- $TTL => Time to live value for the zone.
- $ORIGIN => Defines base name -used in domain name subtitution.
- $INCLUDE => Include a file

$TTL directive must appear at the top of the zone file before the SOA record.
The SOA must be present in a zone file, and defines the domain global values mainly to do with zone transfer.

If an $ORIGIN directive is not defined - BIND synthesizes an $ORIGIN from the zone name in the named.conf file. Ex:

```conf

// named.conf file fragment

zone "example.com" in {
  type master;
  file "pri.example.com";
}
```

$ORIGIN values must be 'qualified' (they end with a 'dot')

$ORIGIN is used in two contexts during zone file processing:
- The symbol `@` forces subtitution of the current (or synthesized) value of $ORIGIN. The `@` symbol is replaced with the current value of $ORIGIN
- The current value of $ORIGIN is added to any 'unqualified' name (any name which does not end in a 'dot')

** Important **
Every time you edit the zone file, increment the serial value before you restart named otherwise BIND won't apply the change to the zone.

#### DNS generic record format

Resource records have two representations. A textual format described in this chapter

```
owner-name | ttl | class | type | type-specific-data
```

Where:
owner-name => owner-name or label of the node in the zone file to which


#### DNS Forward & DNS Reverse

DNS forward => type of DNS request in which a domain name is used to obtain its corresponding IP address.
A forward DNS request is the opposite of a reverse DNS lookup.

DNS reverse => using an IP address to find a domain name.


-----

## LXC (Linux Container) and LXD (LXC Manager)

```bash

sudo apt-get install lxd lxd-client # if not installed yet
sudo lxd init
lxc launch ubuntu:16.04 app01
lxc list
lxc start app01
lxc stop app01
lxc restart app01
lxc pause app01
lxc exec app01 bash

# Rename/move container
lcx move {old} {new}

# Login to container with another user
lxc exec app01 -- sudo --login --user ubuntu

# Or
lxc exec app01 sudo su ubuntu

```


#### Upgrading LXD

```
# Remove old version
sudo apt-get remove --purge lxd lxd-client

# Install using snap
# snap by installed by default on ubuntu 16.04. Otherwise you need to install with `sudo apt-get install snapd`
sudo snap install lxd

```

#### User permissions
```
# Create lxd group is not exist
sudo groupadd --system lxd

# Add user to lxd group
sudo usermod -G lxd -a <username>

# If you wasn't already part of the lxd group
newgrp lxd
```

#### Talk to remote LXD server
```
# Run on the LXD server at least once

# [::] will use 8443 default, or you can specify [::]:8443
lxc config set core.https_address "[::]"
lxc config set core.trust_password some-password

# To talk with remote LXD server
lxc remote add host-a <ip address or DNS>
```

#### Add ssh authorized keys
```
lxc profile edit default

# Add this under 'config' section
config:
  user.user-data: |
    ssh_authorized_keys:
      - ssh-key public@user
```

#### LXD Installation

```bash
sudo apt-get update && sudo apt-get install zfsutils-linux # LXD storage recommendation is using ZSF

sudo lxd init

# Prompt from command above
# Do you want to configure a new storage pool: yes
# Name of the storage backend to use (dir or zfs): zfs
# Create a new zfs pool: yes
# Name of the new zfs pool: lxd
# Would you like to use an existing block device: no
# Size in GB of the new loop device (1GB mininum) [default: 15]: 15
# Would you like LXD to be available over the network: no (if you choose yes then you will able to manage LXD throuhg local computer without SSH)
# Do you want to configure the LXD bridge: yes

```


#### Bash script to delete multiple container at once
```bash
#!/bin/bash
container_prefix=$1
container_list=$(lxc list --columns=n --format=csv | grep $container_prefix)
for container in $container_list
do
  $(lxc delete $container -f 2>&1)
done
```

Save it to some file and make it executable `sudo chmod +x filename`. Use it with `./filename  <container name or container prefix>`

------

## Consul

Service Discovery

#### Install

On Linux:
- Create bash file named `consul.sh` with content like below:

      #!/bin/bash
      apt-get install -y curl unzip

      mkdir -p /var/lib/consul
      mkdir -p /usr/share/consul
      mkdir -p /etc/consul/conf.d

      curl -OL https://releases.hashicorp.com/consul/0.7.5/consul_0.7.5_linux_amd64.zip
      unzip consul_0.7.5_linux_amd64.zip
      mv consul /usr/local/bin/consul

      # Remove commands below if you don't want to install consul web ui
      curl -OL https://releases.hashicorp.com/consul/0.7.5/consul_0.7.5_web_ui.zip
      unzip consul_0.7.5_web_ui.zip -d dist
      mv dist /usr/share/consul/ui



- But make sure the version is up-to-date
- Run with `sudo ./consul.sh`

On macOS:

- Run `brew update && brew install consul`


## GIT

Passing private key when `git clone`: `ssh-agent bash -c 'ssh-add /path/privatekey; git clone your-repo-path'`

#### Rebase (git rebase)

*Rule of thumb*: Don't rebase shared branch


-----

## SAML (Security Assertion Markup Languange), CAS (Central Authentication Service)


#### Directory Services:
- LDAP (Lighweight Directory Access Protocol)
- AD (Active Directory)


#### SAML perks:
1. No need to type in credentials
2. No need to remember and renew passwords
3. No weak passwords



#### SAML (basic flow):
1. User access the app
2. The app will create SAML request to idP (identity provider)
3. idP build SAML response and send back to the app
4. The app verify SAML response from idP and log user in


#### How SAML works:
- User access remote application using a link on an intranet, a bookmark or similar.
- The application identifies the user's origin (by application subdomain, or IP address or similar) and redirects the user back to the identity provider (IdP), asking for authentication.
  This is the authentication request.
- The user either has an existing active browser session with the identity provider or establishes one by logging into the IdP.
- The IdP builds the authentication response in the form of an XML-document containing the user's username or email address, signs it using an X.509 certificate, and
  posts this information to the service provider (SP).
- The SP, which already knows the IdP and has a certificate fingerprint, retrieves the authentication response and validates it using the certificate fingerprint.
- The identity of the user is established and the user is provided with app access.


----


#### CAS (Central Authentication Service)

is: SSO (Single Sign On) for protocol for the web

CAS protocol involves 3 parties:
- A client web browser
- The web application requesting authentication
- The CAS server

** CAS is a protocol, though there is a software that using the same name **


#### CAS Purpose

- CAS allows multi-tier authentication via proxy address.
- Permit user to access multiple application while providing their credentials (such as userid and password) only once.
- Allows web application to authenticate users without gaining access to a user's security credentials such as password.


#### How CAS works
- The client visits an application requiring authentication, the application redirects it to CAS.
- CAS validates the client's authenticity, usually by checking a username and password againts database (such as Kerberos, LDAP or Active Directory)
- If the authentication succeeds, CAS returns the client to the application, passing along a service ticket (a ticket is like some generated number by a network server for a client).
- The application then validates the ticket by contacting CAS over a secure connection and providing its own service identifier and the ticket.
- CAS then gives the application trusted information about whether a particular user has successfully authenticated.


----


## screen

Tips: https://askubuntu.com/questions/325807/arrow-keys-home-end-tab-complete-keys-not-working-in-shell

- Start with `script /dev/null` if message like this appear `Must be connected to a terminal`

- Create new session with name `screen -S sessionname`

- Detach screen with `ctrl + a` then press `d` or `screen -d`

- Reattach screen with `screen -r sessionname`

- Terminate screen with `screen -X quit`. Run this command inside attached screen you want to kill



----


## Kubernetes

POD

kubectl -> CLI from client
API server -> the only one connector to ETCD
scheduler
controller-manager
kubelet -> agent, middlemen
ETCD -> storage to keep states
Proxy

----

## SSH Tips and Tricks

To clone with specifying private ssh keys: `ssh-agent bash -c 'ssh-add /root/.ssh/id_mac_rsa; git clone git@source.golabs.io:novigrad/yggdrasil.git'`. But you must set the private keys permission to `400` with `sudo chown 400 private_key`


## Setup NGROK

- Download from their official website
- Extract `ngrok.zip` to specific folder
- Run `./ngrok auth tokenKey` to add your auth token to your `.ngrok/ngrok.yml` file
- Run `./ngrok help` for more help.
