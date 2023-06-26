# -*- mode: ruby -*-
# vi: set ft=ruby :

vm_ipaddr='192.168.56.2'
ansible_debug=false
tags = ENV['ANSIBLE_TAGS'] || ""

require './argocd-devbox-config.rb' if File.exists?('argocd-devbox-config.rb')

Vagrant.configure("2") do |config|
  config.vm.box = "fedora/38-cloud-base"
  config.vm.hostname = "argocd-dev"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # We use a private (host-only) network to access the box from VScode
  config.vm.network "private_network", ip: vm_ipaddr

  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "argocd-dev"
    vb.memory = "16384"
    vb.cpus = 4
  end
  
  config.ssh.forward_agent = true
  
  # Packages required for setting up ansible and that need to be there before
  # the reboot need to go into our shell provisioner.
  config.vm.provision "shell" do |shell|
    shell.privileged = true
    shell.inline = <<-SHELL
      dnf upgrade -y --refresh
      dnf install -y kernel-modules ansible python3-pip NetworkManager-initscripts-ifcfg-rh
    SHELL
    shell.reboot = true
  end

  # The rest will be installed and configured through Ansible
  config.vm.provision "ansible_local" do |a|
    a.playbook = "argocd-devbox.yaml"
    a.galaxy_role_file = "ansible-requirements.yaml"
    a.galaxy_roles_path = '/home/vagrant/.ansible/collections'
    a.galaxy_command = 'ansible-galaxy collection install -r %{role_file} -p %{roles_path}'
    a.become = true
    a.extra_vars = "config.yaml"
    a.verbose = if ansible_debug == true; "-vvv"; else ""; end
    if tags != ""
      a.tags = tags
    end
  end

end
