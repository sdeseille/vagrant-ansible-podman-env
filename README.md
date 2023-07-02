# Preparing Podman environment with Vagrant and ansible_local provisioner

Almost a year ago, I wrote this article [Preparing Docker environment with Vagrant and ansible_local provisioner](https://dev.to/sdeseille/preparing-docker-environment-with-vagrant-and-ansiblelocal-provisioner-18b8)

Recently, I discovered [Podman](https://github.com/containers/podman), so I decided to git it a try and that's what my article is about. You will see lots of similarities with my previous article. It is volontary. I practice laziness. :p

## Prerequisites

I mentioned that I practice laziness, but don't worry. I won't simply copy and paste the original text. It is more a capitalization of the knowledge I've acquired.

Here's my toolbox with the specified versions used.

- Microsoft Windows: 10 Version 22H2
- Oracle VirtualBox: 6.1.36 r152435 [url](https://download.virtualbox.org/virtualbox/6.1.36/VirtualBox-6.1.36-152435-Win.exe?source=:ow:o:p:nav:mmddyyVirtualBoxHero)
- Vagrant: 2.2.19 [url](https://releases.hashicorp.com/vagrant/2.2.19/vagrant_2.2.19_x86_64.msi)
- Vagrant Box [ubuntu/focal64]: 20220804 [url](https://app.vagrantup.com/ubuntu/boxes/focal64/versions/20220804.0.0)

## Vagrantfile initialisation

Last time, we have use Ubuntu Focal64. A new stable version of Ubuntu is available now. We select that new one named Ubuntu Jammy64.

>vagrant init ubuntu/jammy64

```powershell
PS D:\vagrant_projects\ubuntu_env> vagrant init ubuntu/jammy64
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
PS D:\vagrant_projects\ubuntu_env>
```

### Content of the Vagrantfile

```powershell
PS D:\vagrant_projects\ubuntu_env> type .\Vagrantfile
```

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/jammy64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```
