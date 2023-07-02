# Preparing Podman environment with Vagrant and ansible_local provisioner

Almost a year ago, I wrote this article [Preparing Docker environment with Vagrant and ansible_local provisioner](https://dev.to/sdeseille/preparing-docker-environment-with-vagrant-and-ansiblelocal-provisioner-18b8)

Recently, I discovered [Podman](https://github.com/containers/podman), so I decided to git it a try and that's what my article is about. You will see lots of similarities with my previous article. It is volontary. I practice laziness. :p

## Prerequisites

I mentioned that I practice laziness, but don't worry. I won't simply copy and paste the original text. It is more a capitalization of the knowledge I've acquired.

Here's my toolbox with the specified versions used.

- Microsoft Windows: 10 Version 22H2
- Oracle VirtualBox: 7.0.8 r156879 [url](https://download.virtualbox.org/virtualbox/7.0.8/VirtualBox-7.0.8-156879-Win.exe)
- Vagrant: 2.3.7 [url](https://releases.hashicorp.com/vagrant/2.3.7/vagrant_2.3.7_windows_amd64.msi)
- Vagrant Box [ubuntu/jammy64]: 20230616 [url](https://app.vagrantup.com/ubuntu/boxes/jammy64/versions/20230616.0.0)

## Vagrantfile initialisation

Last time, we used Ubuntu Focal64. A new stable version of Ubuntu is available now. We choose this new version named Ubuntu Jammy64.

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

## Provisioner ansible_local

We don't change what works well. My Desktop environment is still Microsoft Windows. Obviously, we'll be using [**ansible_local**]. As a reminder, when this provisioner is selected, vagrant installs Ansible on the guest system and automaticaly runs your playbook on it.

See the Official [documentation](https://www.vagrantup.com/docs/provisioning/ansible_local).

The minimal configuration for ansible_local is to defined a [**playbook.yml**] file that is read from the current project directory.

```ruby
  # Run Ansible from the Vagrant VM
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
  end
```

Last time, I used Python 2.7 and had some warnings about deprecation of this version. Let see, if I can use Python 3 now. After, some tries and fail, you can find a working version of vagrant provisioner settings below.

```ruby
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.galaxy_role_file = "requirements.yml"
    ansible.galaxy_command = "ansible-galaxy collection install -r %{role_file}"
  end
```

## Minimal validation playbook

As I initialize the environment, my playbook content is very short to start.

```yaml
---
- hosts: all
  become: true
  vars:
    default_container_name: podman
    default_container_image: ubuntu
    default_container_command: sleep 1
  tasks:
    - name: Get uptime information
      ansible.builtin.shell: /usr/bin/uptime
      register: result

    - name: Print return information from the previous task
      ansible.builtin.debug:
        var: result

```

Below, you can see the result.

```powershell
PS D:\vagrant_projects\ubuntu_env> vagrant status
Current machine states:

default                   not created (virtualbox)

The environment has not yet been created. Run `vagrant up` to
create the environment. If a machine is not created, only the
default provider will be shown. So if a provider is not listed,
then the machine is not created for that environment.
PS D:\vagrant_projects\ubuntu_env> vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/jammy64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/jammy64' version '20230616.0.0' is up to date...
==> default: Setting the name of the VM: ubuntu_env_default_1688317157246_80318
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection reset. Retrying...
    default: Warning: Connection aborted. Retrying...
    default: Warning: Connection reset. Retrying...
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default:
    default: Guest Additions Version: 6.0.0 r127566
    default: VirtualBox Version: 7.0
==> default: Mounting shared folders...
    default: /vagrant => D:/vagrant_projects/ubuntu_env
==> default: Running provisioner: ansible_local...
    default: Installing Ansible...
    default: Running ansible-galaxy...
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/containers-podman-1.10.2.tar.gz to /home/vagrant/.ansible/tmp/ansible-local-3302p92f9kmj/tmp410gn7sq/containers-podman-1.10.2-l85_wdjl
Installing 'containers.podman:1.10.2' to '/home/vagrant/.ansible/collections/ansible_collections/containers/podman'
containers.podman:1.10.2 was installed successfully
    default: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [default]

TASK [Get uptime information] **************************************************
changed: [default]

TASK [Print return information from the previous task] *************************
ok: [default] => {
    "result": {
        "changed": true,
        "cmd": "/usr/bin/uptime",
        "delta": "0:00:00.095920",
        "end": "2023-07-02 17:03:12.577458",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2023-07-02 17:03:12.481538",
        "stderr": "",
        "stderr_lines": [],
        "stdout": " 17:03:12 up 3 min,  0 users,  load average: 1.27, 0.81, 0.35",
        "stdout_lines": [
            " 17:03:12 up 3 min,  0 users,  load average: 1.27, 0.81, 0.35"
        ]
    }
}

PLAY RECAP *********************************************************************
default                    : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

PS D:\vagrant_projects\ubuntu_env>
```

You can observe that I installed ansible collection [containers.podman](https://docs.ansible.com/ansible/latest/collections/containers/podman/index.html#plugins-in-containers-podman) during the process of provisioning. It is done with the file [requirements.yml].

```yaml
---
collections:
  - name: containers.podman
```
