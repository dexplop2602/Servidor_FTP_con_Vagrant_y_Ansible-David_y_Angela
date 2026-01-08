Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.hostname = "clienteftp"
  config.vm.network "public_network"
  config.vm.provision "shell", path: "provision_client.sh"
end