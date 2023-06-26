## What's this?

This is a Vagrant box that gives you a complete (albeit *opinionated*) Argo CD development environment. It's based off a recent Fedora version, and comes with all tools set up so you can get started to develop and test Argo CD with ease. Its main purpose is to serve as a box to connect to using VScode's remote feature.

I thought it might be useful to other (emerging) Argo CD contributors as well to help them get started with development.

If you submit enhancement requests, bug reports or pull requests to this repository, please do not expect any of them to be considered. The target audience of this box is mainly myself and my own development workflow. I'll happily accept contributions that I think would be great additions to my workflow, while I will not accept those that I don't see fit. Feel free to clone this box and adapt it to your own workflow, though.

## License

All code within this repository is released to the Public Domain under the [Unlicense](LICENSE).

## Pre-requisites

The box is tested with Vagrant v2.3 using VirtualBox as a provider and Linux as the host. Other providers may or may not work as expected. I haven't tested them.

The bare minimum of RAM for the VM is 8GB. The defaults assigned to the VM in the Vagrantfile are 16GB of compute and 4 CPUs.

## Getting started

Getting the box started is simple. Clone the repository, copy the example configurations, edit them to match your setup and then run `vagrant`.

```shell
$ git clone https://github.com/jannfis/argocd-devbox
$ cd argocd-devbox
$ cp argocd-devbox-config.rb.example argocd-devbox-config.rb
$ vi argocd-devbox-config.rb
$ cp config.yaml.example config.yaml
$ vi config.yaml
$ vagrant up
```

## Configuring the box

Some system aspects are configured in `argocd-devbox-config.rb`. This file will be sourced by the `Vagrantfile`.

Most of the configuration is then done in `config.yaml`, which sets up variables to be used in the Ansible playbook used to provision the box.

Apologies, there's not much documentation but both files should be self-explanatory.

## Configuring local SSH

It's recommended that you make a dedicated configuration entry for the box in your ssh client's configuration:

```
Host argocd-dev
        # Replace the IP address if you re-configured the box's IP address
        Hostname 192.168.56.2
        # Remote username, usually vagrant
        User vagrant
        # Leave enabled for SSH agent forwarding
        ForwardAgent yes
        # For gpg-agent forwarding. Replace <local_user_id> with your local user's uid.
        RemoteForward /run/user/1000/gnupg/S.gpg-agent /run/user/<local_user_id>/gnupg/S.gpg-agent.extra
        # If you recreate the box, its host key will change. We won't check nor store it.
        StrictHostKeyChecking no
        UserKnownHostsFile /dev/null
```

## Required and recommended VScode plugins

On the host, only the [Remote SSH feature](https://code.visualstudio.com/docs/remote/ssh) is required, plus any UI extensions you would like to have (such as the vim editor extension). 

On the remote host (aka in the box), I use (and can recommend) the following plugins:

* [Go](https://marketplace.visualstudio.com/items?itemName=golang.Go)
* [Gitlens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)
* [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)