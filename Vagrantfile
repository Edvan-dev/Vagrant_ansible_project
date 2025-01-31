Vagrant.configure("2") do |config|
    # Definir a box base
    config.vm.box = "debian/bookworm64"
  
    # Configurar a VM
    config.vm.provider "virtualbox" do |vb|
      vb.name = "p01_Edvan"
      vb.memory = 1024
    end
  
    # Configuração de rede
    config.vm.network "private_network", ip: "192.168.57.10"
  
    # Configuração de discos adicionais
    (1..3).each do |i|
      config.vm.provider "virtualbox" do |vb|
        vb.customize ["createhd", "--filename", "disk#{i}.vdi", "--size", 10240]
        vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", i, "--device", 0, "--type", "hdd", "--medium", "disk#{i}.vdi"]
      end
    end
  
    # Provisionamento com Ansible
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisionar.yml"
    end
  end
