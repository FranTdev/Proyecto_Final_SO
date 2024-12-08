# Proyecto_Final_SO

## Instalacion
Primero debemos configurar nuestra clave SSH en el host, lo hacemos con el commando: ```ssh-keygen -t rsa -b 4096```
![WhatsApp Image 2024-12-07 at 15 46 47](https://github.com/user-attachments/assets/69ccc798-57b8-4c3e-9adc-37b42ad8c403)


Iniciamos vagrant con el comando ```vagrant init``` en nuestra consola estando dentro de nuestro archivo, en este caso *Proyecto_Final_SO*
![WhatsApp Image 2024-12-07 at 15 47 50](https://github.com/user-attachments/assets/48e4d920-15bb-4ea9-b50f-f7740c216fb3)

Posteriormente configuramos nuestro archivo ***Vagrantfile***:
```ruby

Vagrant.configure("2") do |config|
  # Configuración para la primera máquina virtual web
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/bionic64"
    web.vm.synced_folder ".", "/vagrant"
    web.vm.network "private_network", type: "static", ip: "192.168.56.101"
    web.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 2
    end

    # Agregar la clave SSH pública en web
    web.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
      s.inline = <<-SHELL
        mkdir -p /home/vagrant/.ssh
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
        chmod 700 /home/vagrant/.ssh
        chmod 600 /home/vagrant/.ssh/authorized_keys

        mkdir -p /root/.ssh
        echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        chmod 700 /root/.ssh
        chmod 600 /root/.ssh/authorized_keys
      SHELL
    end

    # Configuración para permitir el uso de sudo sin contraseña
    web.vm.provision "shell", inline: <<-SHELL
      echo 'vagrant ALL=(ALL) NOPASSWD:ALL' | sudo tee -a /etc/sudoers
    SHELL
  end

  # Configuración para la segunda máquina virtual monitoring
  config.vm.define "monitoring" do |monitoring|
    monitoring.vm.box = "ubuntu/bionic64"
    monitoring.vm.synced_folder ".", "/vagrant"
    monitoring.vm.network "private_network", type: "static", ip: "192.168.56.102"
    monitoring.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 2
    end

    # Agregar la clave SSH pública en monitoring
    monitoring.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
      s.inline = <<-SHELL
        mkdir -p /home/vagrant/.ssh
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
        chmod 700 /home/vagrant/.ssh
        chmod 600 /home/vagrant/.ssh/authorized_keys

        mkdir -p /root/.ssh
        echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        chmod 700 /root/.ssh
        chmod 600 /root/.ssh/authorized_keys
      SHELL
    end

    # Configuración para permitir el uso de sudo sin contraseña
    monitoring.vm.provision "shell", inline: <<-SHELL
      echo 'vagrant ALL=(ALL) NOPASSWD:ALL' | sudo tee -a /etc/sudoers
    SHELL
  end
end
```
