# Mirage Virtualbox VMs via Vagrant

## Setup

### Clone this repo

    $ git clone https://github.com/mirage/mirage-vagrant-vms.git
    $ cd mirage-vagrant-vms

This currently contains support for Ubuntu 14.04 LTS ("Trusty Tahr"), Ubuntu
14.10 ("Utopic Unicorn"), Debian 7.8.0 ("wheezy") and Citrix XenServer 6.5.0.

XenServer support imported from <https://github.com/jonludlam/packer-xenserver>.

### Install `vagrant`

First, install [Vagrant][]. On OSX I use [homebrew][] so I do this as follows:

    $ brew tap phinze/cask
    $ brew install brew-cask
    $ brew cask install vagrant
    $ vagrant --version
    Vagrant 1.4.3

[homebrew]: http://brew.sh/
[vagrant]: http://vagrantup.com/

### Install `packer`

    $ brew tap homebrew/binary
    $ brew install packer

## Use

Build a new box using `packer`:

    $ make box-{ubuntu-14.04,ubuntu-14.10,debian-7.8.0,xenserver-6.5.0}

Bring it up and provision it using `vagrant`:

    $ make vagrant-{ubuntu-14.04,ubuntu-14.10,debian-7.8.0,xenserver-6.5.0}

Connect to it via `ssh`:

    $ make ssh-{ubuntu-14.04,ubuntu-14.10,debian-7.8.0,xenserver-6.5.0}

Finally, within a box, make sure you add `~/bin` to your `$PATH` to use
`0install` installed binaries (specifically, `opam`).

Subsequently, `vagrant halt` will stop the VM (or the usual `shutdown -h now`
when logged into it), `vagrant up` will restart it, and `vagrant ssh` to login.

# _Deprecated_

## Networking

At the risk of this becoming a "general" Xen networking tutorial (as if such a
thing were possible)...


             virtualbox

    [ dom0       ]    [ mirage-domU ]
    [ 100.64.0.2 ]    [ 100.64.0.3  ]
                      [ vifN.0      ]
         |                |
         +---[ xenbr0 ]---+
                 |
                 |
              [ eth2 ]
                 |
                 |


### Useful commands

On host:

    vboxmanage list hostonlyifs
    vboxmanage list dhcpservers

On dom0:

    sudo xm network-list mort-www


### /etc/network/interfaces

network interface configuration. eth0-N, xenbr0

    auto xenbr0
    iface xenbr0 inet dhcp
          bridge_ports eth2
          bridge_stp off
          bridge_waitport 0
          bridge_fd 0

    #VAGRANT-BEGIN
    # The contents below are automatically generated by Vagrant. Do not modify.
    auto eth1
    iface eth1 inet static
          address 172.16.0.2
          netmask 255.255.255.0
    #VAGRANT-END

    #VAGRANT-BEGIN
    # The contents below are automatically generated by Vagrant. Do not modify.
    auto eth2
    iface eth2 inet static
          address 100.64.0.2     <*>
          netmask 255.255.255.0  <*>
    #VAGRANT-END

Lines marked <*> move to the `xenbr0` config from the `eth2` config, and the `eth2` config marked as `manual` not `static`. This makes connection from `dom0` to `mirage-domU` work.

### /etc/dhcp/dhcpd.conf

DHCP server config. subnets from which addresses should be responded with.

    subnet 100.64.0.0 netmask 255.255.255.0 {
      range 100.64.0.2 100.64.0.254;
      # option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
    }
