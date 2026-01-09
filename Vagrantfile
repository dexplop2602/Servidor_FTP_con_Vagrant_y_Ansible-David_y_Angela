# Vagrantfile
Vagrant.configure("2") do |config|
  
  # --- SERVER MACHINE (mirror.sistema.sol) ---
  config.vm.define "server" do |srv|
    srv.vm.box = "debian/bookworm64"
    srv.vm.hostname = "mirror.sistema.sol"
    
    # Bridge Network (Allows connection from other PCs in class)
    srv.vm.network "public_network"

    # Ansible Provisioning (Local execution)
    srv.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "provisioning/playbook.yml"
      ansible.install_mode = "default"
    end
  end

  # --- CLIENT MACHINE (Testing) ---
  config.vm.define "client" do |cli|
    cli.vm.box = "debian/bookworm64"
    cli.vm.hostname = "client"
    cli.vm.network "public_network"

    # Simple script to install testing tools (FTP client, lftp, nmap)
    cli.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y lftp ftp nmap net-tools
      echo "Client machine ready for testing."
    SHELL
  end
end