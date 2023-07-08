# Preparing Podman environment with Vagrant and ansible_local provisioner

![Banner](media/vagrant-ansible-podman-env.drawio.png)

A year ago, I wrote this article [Preparing Docker environment with Vagrant and ansible_local provisioner](https://dev.to/sdeseille/preparing-docker-environment-with-vagrant-and-ansiblelocal-provisioner-18b8)

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

### Minimal validation playbook

As I initialize the environment, my playbook content is very short to start.

```yaml
---
- hosts: all
  become: true
  vars:
    default_container_name: podman
    default_container_image: alpine
    default_container_command: sleep 1
  tasks:
    - name: Get uptime information
      ansible.builtin.shell: /usr/bin/uptime
      register: result

    - name: Print return information from the previous task
      ansible.builtin.debug:
        var: result

```

## First execution of playbook

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

You can observe that I installed ansible collection [containers.podman](https://docs.ansible.com/ansible/latest/collections/containers/podman/index.html#plugins-in-containers-podman) during the process of provisioning. It is done with the file [**requirements.yml**].

```yaml
---
collections:
  - name: containers.podman
```

### playbook for Podman

I used the previous docker example and adapted it to work with podman. The modified ansible playbook is still short.

```yaml
---
- hosts: all
  become: true
  vars:
    default_container_name: podman
    default_container_image: "hello-world"
    default_container_command: sleep 1
  tasks:
    - name: Installing podman
      ansible.builtin.package:
        name: "podman"
        state: present

    - name: Pull an image
      containers.podman.podman_image:
        name: "{{ default_container_image }}"

    - name: Run container
      containers.podman.podman_container:
        name: "{{default_container_name}}"
        image: "{{ default_container_image }}"
        state: started
      register: container_result
  
    - name: Print return information from the previous task
      ansible.builtin.debug:
        var: container_result
```

As you can see, I pull up the well-known docker image [**hello-world**]. After that, I start a container and finally print the content of the registered variable [**container_result**]. In this variable, I am able to retrieve the container dict return by running [**containers.podman.podman_container**] module. The [Official documentation](https://docs.ansible.com/ansible/latest/collections/containers/podman/podman_container_module.html#ansible-collections-containers-podman-podman-container-module) explains this very well.

## Issues when trying to vagrant up

I encountered several issues when executing the command:

>vagrant up

```powershell
PS D:\vagrant_projects\ubuntu_env> vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/jammy64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/jammy64' version '20230616.0.0' is up to date...
==> default: Setting the name of the VM: ubuntu_env_default_1688834174097_92269
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
Timed out while waiting for the machine to boot. This means that
Vagrant was unable to communicate with the guest machine within
the configured ("config.vm.boot_timeout" value) time period.

If you look above, you should be able to see the error(s) that
Vagrant had when attempting to connect to the machine. These errors
are usually good hints as to what may be wrong.

If you're using a custom box, make sure that networking is properly
working and you're able to connect to the machine. It is a common
problem that networking isn't setup properly in these boxes.
Verify that authentication configurations are also setup properly,
as well.

If the box appears to be booting properly, you may want to increase
the timeout ("config.vm.boot_timeout") value.

```

I first tried to increase the value associated to [**config.vm.boot_timeout**] attribute. I set it to [**600**] without success.

The machine worked but the network was not configured and no playbook ran.

```powershell
PS D:\vagrant_projects\ubuntu_env> vagrant status
Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```

If I tried to connet in ssh mode, nothing happened.

```powershell
PS D:\vagrant_projects\ubuntu_env> vagrant ssh

PS D:\vagrant_projects\ubuntu_env> $?
False

```

After examining my laptop settings and searching on the web, I began to suspect something linked with Windows firewall rules.
I exmined my settings and discovered that the network profile for my ethernet interface was set to [**Public**]. I remembered that firewall blocked traffic on this network profile. Since my laptop is at home, I changed the network profile to [**private**]. I destroyed the box that wasn't working.

```powershell
PS D:\vagrant_projects\ubuntu_env> vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...


PS D:\vagrant_projects\ubuntu_env> vagrant status
Current machine states:

default                   not created (virtualbox)

The environment has not yet been created. Run `vagrant up` to
create the environment. If a machine is not created, only the
default provider will be shown. So if a provider is not listed,
then the machine is not created for that environment.
```

After correcting the network type of my ethernet card (from public to private), provisionning worked as expected.

```powershell
PS D:\vagrant_projects\ubuntu_env> vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/jammy64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/jammy64' version '20230616.0.0' is up to date...
==> default: Setting the name of the VM: ubuntu_env_default_1688822696174_94691
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
Downloading https://galaxy.ansible.com/download/containers-podman-1.10.2.tar.gz to /home/vagrant/.ansible/tmp/ansible-local-3338y_lay1dt/tmpu3elz2h1/containers-podman-1.10.2-r9azj1n_
Installing 'containers.podman:1.10.2' to '/home/vagrant/.ansible/collections/ansible_collections/containers/podman'
containers.podman:1.10.2 was installed successfully
    default: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [default]

TASK [Installing podman] *******************************************************
changed: [default]

TASK [Pull an image] ***********************************************************
changed: [default]

TASK [Run container] ***********************************************************
changed: [default]

TASK [Print return information from the previous task] *************************
ok: [default] => {
    "container_result": {
        "actions": [
            "started podman"
        ],
        "changed": true,
        "container": {
            "AppArmorProfile": "containers-default-0.44.4",
            "Args": [
                "/hello"
            ],
            "BoundingCaps": [
                "CAP_CHOWN",
                "CAP_DAC_OVERRIDE",
                "CAP_FOWNER",
                "CAP_FSETID",
                "CAP_KILL",
                "CAP_NET_BIND_SERVICE",
                "CAP_SETFCAP",
                "CAP_SETGID",
                "CAP_SETPCAP",
                "CAP_SETUID",
                "CAP_SYS_CHROOT"
            ],
            "Config": {
                "Annotations": {
                    "io.container.manager": "libpod",
                    "io.kubernetes.cri-o.Created": "2023-07-08T13:28:42.432362679Z",
                    "io.kubernetes.cri-o.TTY": "false",
                    "io.podman.annotations.autoremove": "FALSE",
                    "io.podman.annotations.init": "FALSE",
                    "io.podman.annotations.privileged": "FALSE",
                    "io.podman.annotations.publish-all": "FALSE",
                    "org.opencontainers.image.stopSignal": "15"
                },
                "AttachStderr": false,
                "AttachStdin": false,
                "AttachStdout": false,
                "Cmd": [
                    "/hello"
                ],
                "CreateCommand": [
                    "podman",
                    "container",
                    "run",
                    "--name",
                    "podman",
                    "--detach=True",
                    "hello-world"
                ],
                "Domainname": "",
                "Entrypoint": "",
                "Env": [
                    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                    "TERM=xterm",
                    "container=podman",
                    "HOSTNAME=90b5486779a6"
                ],
                "Hostname": "90b5486779a6",
                "Image": "docker.io/library/hello-world:latest",
                "Labels": null,
                "OnBuild": null,
                "OpenStdin": false,
                "StdinOnce": false,
                "StopSignal": 15,
                "StopTimeout": 10,
                "Timeout": 0,
                "Tty": false,
                "Umask": "0022",
                "User": "",
                "Volumes": null,
                "WorkingDir": "/"
            },
            "ConmonPidFile": "/run/containers/storage/overlay-containers/90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a/userdata/conmon.pid",
            "Created": "2023-07-08T13:28:42.432362679Z",
            "Dependencies": [],
            "Driver": "overlay",
            "EffectiveCaps": [
                "CAP_CHOWN",
                "CAP_DAC_OVERRIDE",
                "CAP_FOWNER",
                "CAP_FSETID",
                "CAP_KILL",
                "CAP_NET_BIND_SERVICE",
                "CAP_SETFCAP",
                "CAP_SETGID",
                "CAP_SETPCAP",
                "CAP_SETUID",
                "CAP_SYS_CHROOT"
            ],
            "ExecIDs": [],
            "ExitCommand": [
                "/usr/bin/podman",
                "--root",
                "/var/lib/containers/storage",
                "--runroot",
                "/run/containers/storage",
                "--log-level",
                "warning",
                "--cgroup-manager",
                "systemd",
                "--tmpdir",
                "/run/libpod",
                "--runtime",
                "crun",
                "--events-backend",
                "journald",
                "container",
                "cleanup",
                "90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a"
            ],
            "GraphDriver": {
                "Data": {
                    "LowerDir": "/var/lib/containers/storage/overlay/01bb4fce3eb1b56b05adf99504dafd31907a5aadac736e36b27595c8b92f07f1/diff",
                    "UpperDir": "/var/lib/containers/storage/overlay/15a506e2bcf599cf6450ec63392af4334501948cddf9f1e5a89d6db5c46826f5/diff",
                    "WorkDir": "/var/lib/containers/storage/overlay/15a506e2bcf599cf6450ec63392af4334501948cddf9f1e5a89d6db5c46826f5/work"
                },
                "Name": "overlay"
            },
            "HostConfig": {
                "AutoRemove": false,
                "Binds": [],
                "BlkioDeviceReadBps": null,
                "BlkioDeviceReadIOps": null,
                "BlkioDeviceWriteBps": null,
                "BlkioDeviceWriteIOps": null,
                "BlkioWeight": 0,
                "BlkioWeightDevice": null,
                "CapAdd": [],
                "CapDrop": [
                    "CAP_AUDIT_WRITE",
                    "CAP_MKNOD",
                    "CAP_NET_RAW"
                ],
                "Cgroup": "",
                "CgroupConf": null,
                "CgroupManager": "systemd",
                "CgroupMode": "private",
                "CgroupParent": "",
                "Cgroups": "default",
                "ConsoleSize": [
                    0,
                    0
                ],
                "ContainerIDFile": "",
                "CpuCount": 0,
                "CpuPercent": 0,
                "CpuPeriod": 0,
                "CpuQuota": 0,
                "CpuRealtimePeriod": 0,
                "CpuRealtimeRuntime": 0,
                "CpuShares": 0,
                "CpusetCpus": "",
                "CpusetMems": "",
                "Devices": [],
                "DiskQuota": 0,
                "Dns": [],
                "DnsOptions": [],
                "DnsSearch": [],
                "ExtraHosts": [],
                "GroupAdd": [],
                "IOMaximumBandwidth": 0,
                "IOMaximumIOps": 0,
                "IpcMode": "private",
                "Isolation": "",
                "KernelMemory": 0,
                "Links": null,
                "LogConfig": {
                    "Config": null,
                    "Path": "",
                    "Size": "0B",
                    "Tag": "",
                    "Type": "journald"
                },
                "Memory": 0,
                "MemoryReservation": 0,
                "MemorySwap": 0,
                "MemorySwappiness": 0,
                "NanoCpus": 0,
                "NetworkMode": "bridge",
                "OomKillDisable": false,
                "OomScoreAdj": 0,
                "PidMode": "private",
                "PidsLimit": 2048,
                "PortBindings": {},
                "Privileged": false,
                "PublishAllPorts": false,
                "ReadonlyRootfs": false,
                "RestartPolicy": {
                    "MaximumRetryCount": 0,
                    "Name": ""
                },
                "Runtime": "oci",
                "SecurityOpt": [],
                "ShmSize": 65536000,
                "Tmpfs": {},
                "UTSMode": "private",
                "Ulimits": [
                    {
                        "Hard": 1048576,
                        "Name": "RLIMIT_NOFILE",
                        "Soft": 1048576
                    },
                    {
                        "Hard": 4194304,
                        "Name": "RLIMIT_NPROC",
                        "Soft": 4194304
                    }
                ],
                "UsernsMode": "",
                "VolumeDriver": "",
                "VolumesFrom": null
            },
            "HostnamePath": "/run/containers/storage/overlay-containers/90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a/userdata/hostname",
            "HostsPath": "/run/containers/storage/overlay-containers/90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a/userdata/hosts",
            "Id": "90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a",
            "Image": "9c7a54a9a43cca047013b82af109fe963fde787f63f9e016fdc3384500c2823d",
            "ImageName": "docker.io/library/hello-world:latest",
            "IsInfra": false,
            "MountLabel": "",
            "Mounts": [],
            "Name": "podman",
            "Namespace": "",
            "NetworkSettings": {
                "Bridge": "",
                "EndpointID": "",
                "Gateway": "",
                "GlobalIPv6Address": "",
                "GlobalIPv6PrefixLen": 0,
                "HairpinMode": false,
                "IPAddress": "",
                "IPPrefixLen": 0,
                "IPv6Gateway": "",
                "LinkLocalIPv6Address": "",
                "LinkLocalIPv6PrefixLen": 0,
                "MacAddress": "",
                "Networks": {
                    "podman": {
                        "DriverOpts": null,
                        "EndpointID": "",
                        "Gateway": "",
                        "GlobalIPv6Address": "",
                        "GlobalIPv6PrefixLen": 0,
                        "IPAMConfig": null,
                        "IPAddress": "",
                        "IPPrefixLen": 0,
                        "IPv6Gateway": "",
                        "Links": null,
                        "MacAddress": "",
                        "NetworkID": "podman"
                    }
                },
                "Ports": {},
                "SandboxID": "",
                "SandboxKey": ""
            },
            "OCIConfigPath": "/var/lib/containers/storage/overlay-containers/90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a/userdata/config.json",
            "OCIRuntime": "crun",
            "Path": "/hello",
            "PidFile": "/run/containers/storage/overlay-containers/90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a/userdata/pidfile",
            "Pod": "",
            "ProcessLabel": "",
            "ResolvConfPath": "/run/containers/storage/overlay-containers/90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a/userdata/resolv.conf",
            "RestartCount": 0,
            "Rootfs": "",
            "State": {
                "Dead": false,
                "Error": "",
                "ExitCode": 0,
                "FinishedAt": "2023-07-08T13:28:43.979040512Z",
                "Healthcheck": {
                    "FailingStreak": 0,
                    "Log": null,
                    "Status": ""
                },
                "OOMKilled": false,
                "OciVersion": "1.0.2-dev",
                "Paused": false,
                "Pid": 0,
                "Restarting": false,
                "Running": false,
                "StartedAt": "2023-07-08T13:28:43.975121172Z",
                "Status": "exited"
            },
            "StaticDir": "/var/lib/containers/storage/overlay-containers/90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a/userdata"
        },
        "failed": false,
        "podman_actions": [
            "podman run --name podman --detach=True hello-world"
        ],
        "podman_systemd": {
            "container-podman": "# container-podman.service\n# autogenerated by Podman 3.4.4\n# Sat Jul  8 13:28:45 UTC 2023\n\n[Unit]\nDescription=Podman container-podman.service\nDocumentation=man:podman-generate-systemd(1)\nWants=network-online.target\nAfter=network-online.target\nRequiresMountsFor=/run/containers/storage\n\n[Service]\nEnvironment=PODMAN_SYSTEMD_UNIT=%n\nRestart=on-failure\nTimeoutStopSec=70\nExecStart=/usr/bin/podman start podman\nExecStop=/usr/bin/podman stop -t 10 podman\nExecStopPost=/usr/bin/podman stop -t 10 podman\nPIDFile=/run/containers/storage/overlay-containers/90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a/userdata/conmon.pid\nType=forking\n\n[Install]\nWantedBy=default.target\n"
        },
        "stderr": "",
        "stderr_lines": [],
        "stdout": "90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a\n",
        "stdout_lines": [
            "90b5486779a6a9c5780a76a985da10da2c0179acb8410ef2ce77f8b806cf978a"
        ]
    }
}

PLAY RECAP *********************************************************************
default                    : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

PS D:\vagrant_projects\ubuntu_env>
```

## But... it's still not enough

I systematically check that my hypotheses are correct and reproducible. I try again to destroy and up my box after that. The symptom reappears. I spend more time searching and trying to understand what's wrong. In fact, once I've destroyed the box, if I up it immediately afterwards, the problem occurs. If I wait a bit before Up it, everything's fine. I found an [old vagrant issue in github](https://github.com/hashicorp/vagrant/issues/8846) mentioning my symptom and someone suggested the following command to solve this problem.

>netsh winsock reset

The command must be executed in cmd or powershell as administrator.

I found others articles about the winsock issue in Windows 10.

- <https://www.minitool.com/news/winsock-reset-command-windows-10.html>
- <https://www.howtogeek.com/785351/how-and-why-to-perform-a-netsh-winsock-reset-on-windows/>

And indeed, after running the command, I can up my boxe after destroying it. So if you are patient enough, you wait a little while before recreating the boxe after destroying it, or you use the command right after destroying the box.

## connecting to provisionned boxe

You use following command to connect to vagrant box.

>vagrant ssh

```powershell
PS D:\vagrant_projects\ubuntu_env> vagrant ssh
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-75-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Jul  8 19:59:04 UTC 2023

  System load:  0.3203125         Processes:                    105
  Usage of /:   5.7% of 38.70GB   Users logged in:              0
  Memory usage: 16%               IPv4 address for cni-podman0: 10.88.0.1
  Swap usage:   0%                IPv4 address for enp0s3:      10.0.2.15


Expanded Security Maintenance for Applications is not enabled.

20 updates can be applied immediately.
16 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


vagrant@ubuntu-jammy:~$
```

## Executing podman with sudo

You can use following command to run the hello-world container example with sudo.

>sudo podman run hello-world

```bash
vagrant@ubuntu-jammy:~$ sudo podman run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

## Executing podman as user

You can also use following command to run the hello-world container example as unprivileged user.

>podman run hello-world


```bash
vagrant@ubuntu-jammy:~$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant)

vagrant@ubuntu-jammy:~$ podman run hello-world
Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/shortnames.conf)
Trying to pull docker.io/library/hello-world:latest...
Getting image source signatures
Copying blob 719385e32844 done
Copying config 9c7a54a9a4 done
Writing manifest to image destination
Storing signatures

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

## Conclusion

Replacing docker with podman is interesting. The fact, that podman doesn't require a daemon to run makes it easier to use with unprivileged user profiles. The tool's syntax makes it easy to make the transition from docker. In fact, it's often enough to replace docker with podman in the command line, retaining all the arguments inherited from docker, and it works.

All files are available from my GitHub Account [Here](https://github.com/sdeseille/vagrant-ansible-podman-env).
I'll be working in more depth with podman and will certainly write an article on using podman in rootless mode.
