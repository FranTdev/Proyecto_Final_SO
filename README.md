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

Ahora arrancamos las maquinas virtuales atraves del comando ```vagrant up```
![WhatsApp Image 2024-12-07 at 15 55 35](https://github.com/user-attachments/assets/e3863f11-6e03-41bd-921d-8150c423f3f2)

Podemos confirmar el estado de las maquinas haciendo uso de ```vagrant status```
![WhatsApp Image 2024-12-07 at 16 04 58](https://github.com/user-attachments/assets/a767e385-4eb3-471d-8c3f-378e56d17740)

Y nos conectamos a nuestra maquina ***web*** con el comando ```vagrant ssh web```
![WhatsApp Image 2024-12-07 at 16 06 56](https://github.com/user-attachments/assets/e350d3fb-1ad7-430c-b01c-097586fc3bcb)

Despues de realizar estos pasos llego el momento de actualizar nuestro instalador de paquetes apt y de instalar ansible en nuestra maquina:
```bash
sudo apt update
sudo apt install ansible -y
```
![WhatsApp Image 2024-12-07 at 16 10 41](https://github.com/user-attachments/assets/4528e63b-5f60-44f4-91b1-ff03e2c0fb37)

![WhatsApp Image 2024-12-07 at 16 11 19](https://github.com/user-attachments/assets/b24b36e7-95ae-4a79-b691-3ce867d6f654)

Procedemos a mover las claves privadas de las diferentes maquinas y cambiar sus permisos:
```bash
cp /vagrant/.vagrant/machines/vm1/virtualbox/private_key /home/vagrant/.ssh/private_key_web
chmod 600 /home/vagrant/.ssh/private_key_web
cp /vagrant/.vagrant/machines/vm2/virtualbox/private_key /home/vagrant/.ssh/private_key_monitoring
chmod 600 /home/vagrant/.ssh/private_key_monitoring
```
![WhatsApp Image 2024-12-07 at 16 48 50](https://github.com/user-attachments/assets/a46b2a2e-c38b-4b59-a837-2584b684cb13)

![WhatsApp Image 2024-12-07 at 16 49 30](https://github.com/user-attachments/assets/7ab15d4b-455d-4d6c-8c4e-37aa40c1db5a)

Ahora creamos nuestro archivo inventory o host:
![WhatsApp Image 2024-12-07 at 16 51 55](https://github.com/user-attachments/assets/e3762e10-0ef1-479c-be47-4fc74e284155)

Ahora solo debemos crear y ejecutar los playbooks y nos ayudaremos de la herramienta de nano:               

Para NGINX cremos un archivo nginx.yml
```yml
[Uploading nginx.yml…]()---
- name: Ensure Python is installed
  hosts: all
  gather_facts: no 
  become: yes
  tasks:
    - name: Install Python
      raw: sudo apt update && sudo apt install -y python3
    - name: Create symlink for Python
      raw: sudo ln -sf /usr/bin/python3 /usr/bin/python


- name: Instalar Nginx en Web
  hosts: vm1
  become: yes
  tasks:
    - name: Actualizar caché de apt
      apt:
        update_cache: yes

    - name: Instalar Nginx
      apt:
        name: nginx
        state: present

    - name: Iniciar y habilitar Nginx
      service:
        name: nginx
        state: started
        enabled: yes
```
Para descargar e instalar prometeus y grafana hacemos uso del archivo que llamaremos ***prometeus_grafana.yml***

```yml
[Uploading prometeus_grafana.yml…]()---
- name: Instalar Prometheus y Grafana en monitoring
  hosts: monitoring
  become: yes
  tasks:
    - name: Actualizar caché de apt
      apt:
        update_cache: yes

    - name: Instalar Prometheus
      apt:
        name: prometheus
        state: present

    - name: Agregar la clave del repositorio de Grafana
      apt_key:
        url: "https://packages.grafana.com/gpg.key"
        state: present

    - name: Agregar el repositorio de Grafana
      apt_repository:
        repo: "deb https://packages.grafana.com/oss/deb stable main"
        state: present
        filename: grafana

    - name: Actualizar caché después de agregar el repositorio de Grafana
      apt:
        update_cache: yes

    - name: Instalar Grafana
      apt:
        name: grafana
        state: present

    - name: Iniciar y habilitar Prometheus
      service:
        name: prometheus
        state: started
        enabled: yes

    - name: Iniciar y habilitar Grafana
      service:
        name: grafana-server
        state: started
        enabled: yes

```
Ejecutamos los comandos
```bash
ansible-playbook -i /vagrant/hosts /vagrant/install_nginx.yml
ansible-playbook -i /vagrant/hosts /vagrant/install_prometheus_grafana.yml
```
![WhatsApp Image 2024-12-07 at 21 12 23](https://github.com/user-attachments/assets/24e5940a-6f22-4d90-9b9b-5645e1af870f)

##Confirmamos el funcionamiento de estos servicios ingresando las ip correspondiente en nuestro navegador.


![WhatsApp Image 2024-12-07 at 21 12 23 (2)](https://github.com/user-attachments/assets/6bfecf45-f7ab-4848-ba6f-d85c8220b44f)

![WhatsApp Image 2024-12-07 at 21 14 00](https://github.com/user-attachments/assets/c32d5219-8096-4a76-a4e1-a062753911ea)

![WhatsApp Image 2024-12-07 at 21 12 23 (1)](https://github.com/user-attachments/assets/b3ba189f-2403-4dea-b858-b82d113673e5)



