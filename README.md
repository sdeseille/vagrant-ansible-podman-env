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

