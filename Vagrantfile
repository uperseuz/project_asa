# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

#Box padrão de todas as vms
  config.vm.box = "debian/bookworm64"

  # Não gerar novas chaves SSH
  config.ssh.insert_key = false

  # Desabilitar guest additions
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

  # Configurações do provedor VirtualBox:
  config.vm.provider :virtualbox do |v|
    v.memory = 512           # Memória RAM padrão as vms
    v.linked_clone = true    # Ativa linked clone
    v.check_guest_additions = false  # Desabilita verificação de guest additions
  end



  # Trigger para desabilitar DHCP do VirtualBox (Host-Only)
  config.trigger.before :"Vagrant::Action::Builtin::WaitForCommunicator", type: :action do |t|
    t.warn = "Iterrompe o servidor dhcp do virtualbox"
    t.run = {inline: "vboxmanage dhcpserver stop --interface vboxnet0"}
  end
  config.vm.provision "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.become = true
    ansible.playbook = "playbooks/common.yml"
    ansible.groups = {
      "all" => ["arq", "db", "app", "cli"]
  }
  end

  # ================= SERVIDOR DE ARQUIVOS =================
  config.vm.define "arq" do |arq|
    arq.vm.hostname = "arq.caua.joao.devops"
    arq.vm.network "private_network", ip: "192.168.56.121"

    # Criar e anexar 3 discos de 10GB
    (0..2).each do |x|
    arq.vm.disk :disk, size: "10GB", name: "disk-#{x}"
  end


    arq.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/arq.yml"
      ansible.become = true
      ansible.verbose = "v"
    end
  end

  # ================= SERVIDOR DB =================
  config.vm.define "db" do |db|
    db.vm.hostname = "db.caua.joao.devops"

    db.vm.network "private_network",
      type: "dhcp",
      mac: "0800271A0021"

    db.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/db.yml"
      ansible.become = true
      ansible.verbose = "v"
    end
  end

  # ================= SERVIDOR APP =================
  config.vm.define "app" do |app|
    app.vm.hostname = "app.caua.joao.devops"

    app.vm.network "private_network",
      type: "dhcp",
      mac: "0800272B2100"

    app.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/app.yml"
      ansible.become = true
      ansible.verbose = "v"
    end
  end

  # ================= CLIENTE =================
  config.vm.define "cli" do |cli|
    cli.vm.hostname = "cli.caua.joao.devops"
    cli.vm.network "private_network", type: "dhcp"

    cli.vm.provider "virtualbox" do |vb|
      vb.name = "cli-devops"
      vb.memory = 1024
    end

    cli.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/cli.yml"
      ansible.become = true
      ansible.verbose = "v"
    end
  end
end
