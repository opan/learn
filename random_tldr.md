VPN = Virtual Private Network
IKE = Internet Key Exchange (v1/v2)
  => protocol used to set up a security association (SA) in the IPsec protocol suite.
  => protocol that allows for direct IPsec tunneling between the server and client.


IPsec = Internet Protocol Security (IPsec)
  => a network protocol suite that authenticates and encrypts the packets of data sent over a network.

StrongSwan => an open source IPsec daemon


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
