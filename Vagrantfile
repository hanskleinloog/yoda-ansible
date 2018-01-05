# coding: utf-8
# copyright Utrecht University
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Retrieve instance from command line.
require 'getoptlong'

opts = GetoptLong.new(
  [ '--instance', GetoptLong::OPTIONAL_ARGUMENT ]
)

instance='combined'
opts.each do |opt, arg|
  case opt
    when '--instance'
      instance=arg
  end
end

# Configuration variables.
VAGRANTFILE_API_VERSION = "2"

BOX = 'centos/7'
GUI = false
CPU = 1
RAM = 512

DOMAIN  = ".yoda.test"
NETWORK = "192.168.50."
NETMASK = "255.255.255.0"

if instance == "full" then
  HOSTS = {
    "portal"   => [NETWORK+"10", CPU, RAM, GUI, BOX],
    "database" => [NETWORK+"11", CPU, RAM, GUI, BOX],
    "icat"     => [NETWORK+"12", CPU, RAM, GUI, BOX],
    "resource" => [NETWORK+"13", CPU, RAM, GUI, BOX],
    "public"   => [NETWORK+"14", CPU, RAM, GUI, BOX],
  }
else
  HOSTS = {
    "combined" => [NETWORK+"10", 2, 1024, GUI, BOX],
    "ldap"     => [NETWORK+"11", CPU, RAM, GUI, BOX],
    "comanage" => [NETWORK+"12", CPU, RAM, GUI, BOX],    
  }
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.insert_key = false

  HOSTS.each do | (name, cfg) |
    ipaddr, cpu, ram, gui, box = cfg

    config.vm.define name do |machine|
      machine.vm.box = box

      machine.vm.provider "virtualbox" do |vbox|
        vbox.gui    = gui
        vbox.cpus   = cpu
        vbox.memory = ram
        vbox.name   = name
        vbox.customize ["guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 10000]
      end

      machine.vm.hostname = name + DOMAIN
      machine.vm.network 'private_network', ip: ipaddr, netmask: NETMASK
      machine.vm.synced_folder ".", "/vagrant", disabled: true
    end
  end

  # Provision controller for Ansible on Windows host.
  if Vagrant::Util::Platform.windows? then
    config.vm.define "controller" do |controller|
      controller.vm.provider "virtualbox" do |vbox|
        vbox.customize ["guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 10000]
      end
      controller.vm.box = BOX
      controller.vm.hostname = "controller"
      controller.vm.network :private_network, ip: "192.168.50.5", netmask: NETMASK
      controller.vm.provision "shell", privileged: false, path: "vagrant/provision_controller.sh"
      controller.vm.synced_folder ".", "/vagrant", disabled: true
    end
  end
end
