Title: Configuring for Vagrant


# Configuring for Vagrant

The Juju Vagrant images are a way of extending the portability and ease of use
of LXC containers to other platforms. The initial image is based on the Ubuntu
Cloud Images. Juju is configured to use the local LXC provider and is preseeded
with the latest LTS releases so deployments should be relatively fast.

Using these images gives you a couple of things:

- A Vagrant environment optimized for charm development.
- The Juju GUI accessible locally through your browser.
- A functional, self-contained Juju environment.

In short, enough of an environment to write and test charms or sandbox your
Juju deployments.


## Installing

Ensure the following software components are installed on your development
machine:

^# Ubuntu

  ```bash
  sudo apt-get update
  sudo apt-get -y install virtualbox vagrant
  ```
  !!! Document the route setup to replace sshuttle

^# Mac OS X
  1. Fetch and install VirtualBox from [virtualbox.org](https://www.virtualbox.org/)

  - [Homebrew](http://brew.sh)
  - [Vagrant](https://www.vagrantup.com)
  - [VirtualBox](https://www.virtualbox.org/)
  2. Install Vagrant from [vagrantup.com](https://www.vagrantup.com/downloads.html)

  !!! This workflow can be modified to use brew entirely:
  install brew
  brew install cask
  brew cask install virtualbox
  brew cask install vagrant

  !!! NOTE: XCode used to be required. After a certain version, brew became
  able to download just the command line tools without the whole 700M+ install.
  Note the version of OS X that requires Xcode prior to installing brew.

^# Windows
  - [Vagrant](https://www.vagrantup.com)
  - [VirtualBox](https://www.virtualbox.org/)

  1. Fetch and install VirtualBox from [virtualbox.org](https://www.virtualbox.org/)
  2. Install Vagrant from [vagrantup.com](https://www.vagrantup.com/downloads.html)

## Choosing a Vagrant Image

We recommend using the [trusty64-juju] box, which is the official 64-bit version of Ubuntu 14.04 LTS, aka Trusty Tahr, with Juju included.

If you prefer a different version of Ubuntu, you can find more options available in the [Vagrant Cloud].


## Getting started!

Vagrant makes getting started really easy.

Create a directory to work in. This directory will be shared with the guest,
and contain the vagrant configuration for the machine.

Make a working directory for working with Vagrant. This will be where `Vagrantfile` lives, as well as the charm you are developing.

```bash
mkdir ~/vagrant
cd ~/vagrant
```

## System Requirements

!!! TODO: Need to recommend minimum and recommended requirements for running
the VM (memory and disk, specifically). How much space does the downloaded
image actually use, and what may happen if you do run out (like when you're
in the middle of doing a demo).

## Initialize

Initialize this environment by running:

```bash
vagrant init ubuntu/trusty64-juju
```


### Configuring Vagrantfile

The Vagrantfile is filled with configuration options. We'll specifically note the most common ones you may want to adjust.

!!! Note: By default, Vagrant boxes based on the Ubuntu cloud images are configured to use
2GB of memory by default. This is suitable for most charm development, but
large charms (>500MB) may require additional memory. As a general rule of thumb, allocate an
additional 1GB of memory for every 500MB of disk space that your charm
requires.  For example, a 500MB charm may need a Vagrant image with 3GB of
memory; a 1GB charm may need 4GB; etc.

#### Memory

Increasing the amount of memory available to the virtual machine will
improve its performance. Beware allocating too much memory, though, or your system performance will suffer.


```ruby
config.vm.provider :virtualbox  do |vb|
  # Increase the memory to 4G
  mem = 1024 * 4
  vb.customize ["modifyvm", :id, "--memory", mem]
end
```

#### CPU

```ruby
config.vm.provider :virtualbox  do |vb|
  # Increase the memory to 4G
  mem = 1024 * 4
  vb.customize ["modifyvm", :id, "--memory", mem]
end
```

#### Networking

If you want to connect to a service running within the virtual machine, there are a few additional steps that you'll need to take.

First, install the vagrant-triggers plugin. This will allow you to automate

```bash
vagrant plugin install vagrant-triggers
```

^# Mac OS X/Linux

  To avoid having to type your password every time you start or stop your
  virtual machine, you can add the route command to /etc/sudoers:

  ```
  youruser ALL=(ALL) NOPASSWD: /sbin/route
  ```

  !!! Note: This should only be used on your development machine, as this will
  give your account the ability to modify your network routing.

  ```ruby
  config.trigger.after [:provision, :up, :reload] do
    system('sudo route add -net 10.0.3.0/24 172.16.250.15 >/dev/null')
  end

  config.trigger.after [:halt, :destroy] do
    system('sudo route delete -net 10.0.3.0/24 172.16.250.15 >/dev/null')
  end
  ```

^# Windows

  !!! TODO: This needs to be tested.

  ```ruby
  config.trigger.after [:provision, :up, :reload] do
    system('route add 10.0.3.0 netmask 255.255.255.0 172.16.250.15 >NUL')
  end

  config.trigger.after [:halt, :destroy] do
    system('route delete 10.0.3.0 >NUL')
  end
  ```


## Routing local traffic to Vagrant

!!! **Note:** If your local network is using 10.0.3.x you will need to alter the
Juju networking in the vagrant box, and substitute the network provided in the
command above


## Vagrant lifecycle

To power on the machine, run `vagrant up`

To SSH in, run `vagrant ssh`

To shut down the machine, run `vagrant halt`

To destroy it, run `vagrant destroy`


## Upgrading your virtual machine

To see if the base virtual machine image is current, run `vagrant box outdated`:

```bash
Checking if box 'ubuntu/trusty64-juju' is up to date...
A newer version of the box 'ubuntu/trusty64-juju' is available! You currently
have version '14.04'. The latest is version '20150911.0.0'. Run
`vagrant box update` to update.
```

To update the base image, run `vagrant box update`:

```bash
==> default: Checking for updates to 'ubuntu/trusty64-juju'
    default: Latest installed version: 14.04
    default: Version constraints:
    default: Provider: virtualbox
==> default: Updating 'ubuntu/trusty64-juju' with provider 'virtualbox' from version
==> default: '14.04' to '20150911.0.0'...
==> default: Loading metadata for box 'https://vagrantcloud.com/ubuntu/trusty64-juju?access_token=yThj18KMfiBiSaqqsboNQoK-unRRA-7zr2sz4Gyd84sbBaMTYqc66ALsn-mx2rs7udg'
==> default: Adding box 'ubuntu/trusty64-juju' (v20150911.0.0) for provider: virtualbox
    default: Downloading: https://atlas.hashicorp.com/ubuntu/boxes/trusty64-juju/versions/20150911.0.0/providers/virtualbox.box
==> default: Successfully added box 'ubuntu/trusty64-juju' (v20150911.0.0) for 'virtualbox'!
```

!!! Disclaimer: In order to take advantage of the updated box, you would need
to `vagrant destroy` and `vagrant up` the new image, which will destroy any
changes you've made inside the environment.


The box will start and configure Juju for you.

![Step two](./media/config-vagrant-step02.png)

## Access the GUI

In order to access the Juju GUI, go to: [http://127.0.0.1:6080](http://127.0.0.1:6080).
It may take some time for it to be deployed so you may need to refresh the page
a few times. Once deployed, the GUI will display the password on the login page.

![Step three](./media/config-vagrant-step03.png)

The GUI is only accessible from your local host. From there you will be able to
deploy charms, add relations, etc.


## Reporting issues with the Vagrant Image

If you encounter any bugs with the Vagrant images, please
[file a bug] report.

[file a bug]: https://bugs.launchpad.net/juju-vagrant-images
[Vagrant Cloud]: https://vagrantcloud.com/ubuntu
[trusty64-juju]: https://vagrantcloud.com/ubuntu/boxes/trusty64-juju
[Homebrew]: http://brew.sh
[Vagrant]: https://www.vagrantup.com
[VirtualBox]: https://www.virtualbox.org/
