## What's this?

This is a Vagrant box that gives you a complete (albeit *opinionated*) Argo CD development environment. It's based off a recent Fedora version, and comes with all tools set up so you can get started to develop and test Argo CD with ease. Its main purpose is to serve as a box to connect to using VScode's remote feature.

I thought it might be useful to other (emerging) Argo CD contributors as well to help them get started with development.

If you submit enhancement requests, bug reports or pull requests to this repository, please do not expect any of them to be considered. The target audience of this box is mainly myself and my own development workflow. I'll happily accept contributions that I think would be great additions to my workflow, while I will not accept those that I don't see fit. Feel free to clone this box and adapt it to your own workflow, though.

## What's in it?

### Go SDKs

There's various Go SDKs in `~/sdk` plus the latest Go version from Fedora.

### Argo CD & toolchain

The provisioning of the box includes cloning the Argo CD repository from your own fork and setting up the required Argo CD development toolchain. Please make sure that the default branch of your fork is up-to-date with the lastest code in Argo CD's default branch, so that your toolchain will be up-to-date after provisioning.

Instead of Docker, the box installs `podman` with the `podman-docker` CLI.

After provisioning, you'll be able to run things like `make test-local`, `make start-e2e-local` and `make codegen` out-of-the-box.

The Argo CD source code will be cloned into `~/go/src/github.com/argoproj/argo-cd`. For convenience, an alias `cdargo` is setup in the login shell that will switch to that directory.

### Git configuration

Git will be configured with a provided username, email address and (optionally) your PGP configuration. The provisioner will add any public GnuPG keys you drop into `files/gpg_public_keys` to the GnuPG keyring on the box and set up everything for forwarding `gpg-agent` connections through SSH.

In the box's configuration file, you can specify additional repositories to be cloned into the box, along with any remote(s) that should be set-up for them.

### Kubernetes

The box installs the stable version of `microk8s` via snap and sets up the box's user's kubeconfig accordingly. Additionally, tools such as `kubens` and `kubectx` will be installed for purposes of convenience.

## License

All code within this repository is released to the Public Domain under the [Unlicense](LICENSE).

## Pre-requisites

The box is tested with Vagrant v2.3 using VirtualBox as a provider and Linux as the host. Other providers may or may not work as expected. I haven't tested them.

The bare minimum of RAM for the VM is 8GB. The defaults assigned to the VM in the Vagrantfile are 16GB of compute and 4 CPUs.

## Getting started

Getting the box started is simple. Clone the repository, copy the example configurations, edit them to match your setup and then run `vagrant`. If you have SSH or GPG keys you would like to include, copy them before starting the box.

```shell
$ git clone https://github.com/jannfis/argocd-devbox
$ cd argocd-devbox
$ cp argocd-devbox-config.rb.example argocd-devbox-config.rb
$ vi argocd-devbox-config.rb
$ cp config.yaml.example config.yaml
$ vi config.yaml
$ # optional, copy your public key to files/ssh_public_keys
$ cp ~/.ssh/id_rsa.pub files/ssh_public_keys/
$ # optional, export your public pgp key to files/gpg_public_keys
$ gpg -a --export mykeyid > files/gpg_public_keys/yourname.asc
$ vagrant up
```

## Configuring the box

Some system aspects are configured in `argocd-devbox-config.rb`. This file will be sourced by the `Vagrantfile`.

Most of the configuration is then done in `config.yaml`, which sets up variables to be used in the Ansible playbook used to provision the box.

Apologies, there's not much documentation but both files should be self-explanatory.

### Public GnuPG keys

All files in `files/gpg_public_keys` that match the pattern `*.asc` will be imported into the box's user's public GnuPG keyring.

### Public SSH keys

All files in `files/ssh_public_keys` that match the pattern `*.pub` will be imported into the box's user's `~/.ssh/authorized_keys`

## Configuring local SSH on the host

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

## Starting, stopping, restarting the box

You can start the box by running the following command in the directory where you checked out this repository:

```
$ vagrant up
```

Likewise, you can stop it by running

```
$ vagrant halt
```

And finally, to restart the box:

```
$ vagrant reload
```

## Re-provisioning the box

If configuration has changed, you can re-provision the box without the need for recreating it. To re-provision it, run

```
$ vagrant --provision --with-provisioner "ansible_local"
```

Note that not all configuration is idempotent (yet), but it should be a non-destructive action (for example, if you remove a repository from the configuration, this repository will not be removed from the box).

## Deleting and re-creating the box

You can delete the box by running

```
$ vagrant destroy
```

and then re-create by running

```
$ vagrant up
```

However, keep in mind that all data within the box will be lost upon `vagrant destroy`. You should make sure that you have commited all your code changes and pushed them to a branch in your fork, otherwise, your changes will be gone.

## Required and recommended VScode plugins

On the host, only the [Remote SSH feature](https://code.visualstudio.com/docs/remote/ssh) is required, plus any UI extensions you would like to have (such as the vim editor extension). 

On the remote host (aka in the box), I use (and can recommend) the following plugins:

* [Go](https://marketplace.visualstudio.com/items?itemName=golang.Go)
* [Gitlens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)
* [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)