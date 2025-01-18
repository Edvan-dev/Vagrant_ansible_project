Vagrant.configure("2") do |config|
    # Define the base box
    config.vm.box = "generic/debian12"
  
    # Set up the VM configuration
    config.vm.provider "virtualbox" do |vb|
      vb.name = "p01_Edvan"
      vb.memory = 1024
    end
  
    # Network configuration
    config.vm.network "private_network", ip: "192.168.57.10"
  
    # Additional disk configuration
    (1..3).each do |i|
      config.vm.provider "virtualbox" do |vb|
        vb.customize ["createhd", "--filename", "disk#{i}.vdi", "--size", 10240]
        vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", i, "--device", 0, "--type", "hdd", "--medium", "disk#{i}.vdi"]
      end
    end
  
    # Provisioning with Ansible
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisionar.yml"
    end
  end
