# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  
  # Configuración global de la red
  # config.vm.network "private_network", type: "dhcp"

  # Nodo 1: Servidor web con Consul
  config.vm.define "appServer1" do |appServer1|
    appServer1.vm.box = "bento/ubuntu-22.04"
    appServer1.vm.hostname = "appServer1"
    appServer1.vm.network "private_network", ip: "192.168.50.13"
    appServer1.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y apache2
      echo "<h1>Hello World..! From appServer1</h1>" | sudo tee /var/www/html/index.html
      sudo systemctl restart apache2
      sudo apt-get install -y nodejs
      sudo apt-get install -y npm
      wget https://releases.hashicorp.com/consul/1.14.1/consul_1.14.1_linux_arm64.zip
      sudo apt install unzip
      unzip consul_1.14.1_linux_arm64.zip
      sudo mv consul /usr/local/bin/
      sudo mkdir /etc/consul.d
      sudo mkdir -p /opt/consul
      sudo chown -R vagrant:vagrant /opt/consul
    SHELL
  end

  # Nodo 2: Servidor web con Consul
  config.vm.define "appServer2" do |appServer2|
    appServer2.vm.box = "bento/ubuntu-22.04"
    appServer2.vm.hostname = "appServer2"
    appServer2.vm.network "private_network", ip: "192.168.50.14"
    appServer2.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y apache2
      echo "<h1>Hello World..! From appServer2 :)</h1>" | sudo tee /var/www/html/index.html
      sudo systemctl restart apache2
      sudo apt-get install -y nodejs
      sudo apt-get install -y npm
      wget https://releases.hashicorp.com/consul/1.14.1/consul_1.14.1_linux_arm64.zip
      sudo apt install unzip
      unzip consul_1.14.1_linux_arm64.zip
      sudo mv consul /usr/local/bin/
      sudo mkdir /etc/consul.d
      sudo mkdir -p /opt/consul
      sudo chown -R vagrant:vagrant /opt/consul
    SHELL
  end

  # Nodo 3: HAProxy y Consul
  config.vm.define "haproxyBalancer" do |haproxyBalancer|
    haproxyBalancer.vm.box = "bento/ubuntu-22.04"
    haproxyBalancer.vm.hostname = "haproxyBalancer"
    haproxyBalancer.vm.network "private_network", ip: "192.168.50.15"
    haproxyBalancer.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y haproxy
      wget https://releases.hashicorp.com/consul/1.14.1/consul_1.14.1_linux_arm64.zip
      sudo apt install unzip
      unzip consul_1.14.1_linux_arm64.zip
      sudo mv consul /usr/local/bin/
      sudo mkdir /etc/consul.d
      sudo mkdir -p /opt/consul
      sudo chown -R vagrant:vagrant /opt/consul

      # Error Personalizado
      echo -e "HTTP/1.0 503 Service Unavailable\nCache-Control: no-cache\nConnection: close\nContent-Type: text/html\r\n\r\n<html><body><h1>503 Service Unavailable</h1>\nSorry, the service is currently unavailable. Please try again later.</body></html>" > /etc/haproxy/errors/503.http

      # Configuración de HAProxy
      echo "
      backend web-backend
        balance roundrobin
        stats enable
        stats auth admin:admin
        stats uri /haproxy?stats
        server app1 192.168.50.13:80 check
        server app2 192.168.50.14:80 check
        errorfile 503 /etc/haproxy/errors/503.http       

        server node1-3000 192.168.50.13:3000 check
        server node1-3001 192.168.50.13:3001 check
        server node1-3002 192.168.50.13:3002 check

        server node2-3000 192.168.50.14:3000 check
        server node2-3001 192.168.50.14:3001 check
        server node2-3002 192.168.50.14:3002 check

      frontend http
        bind *:80
        default_backend web-backend

      " | sudo tee -a /etc/haproxy/haproxy.cfg

      # Reiniciar HAProxy
      sudo systemctl restart haproxy
    SHELL
  end
end