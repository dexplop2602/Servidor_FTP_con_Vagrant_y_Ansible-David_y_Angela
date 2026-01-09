# coding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  # FTP SERVER
  config.vm.define "servidor" do |srv|
    srv.vm.hostname = "servidorftp"
    srv.vm.network "public_network"
    srv.vm.provision "shell", path: "provision.sh"
  end

  # CLIENT
  config.vm.define "cliente" do |cli|
    cli.vm.hostname = "clienteftp"
    cli.vm.network "public_network"
    cli.vm.provision "shell", path: "provision_client.sh"
  end

end