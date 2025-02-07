$geonode_source_path='.'

$script1 = <<-'SCRIPT'
#!/bin/bash

export DEBIAN_FRONTEND="noninteractive"

if [ ! -f '/usr/local/bin/docker-compose' ]; then
    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
fi

sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get -y install docker-ce docker-ce-cli containerd.io pwgen python3-pip python3-virtualenv virtualenvwrapper
sudo adduser vagrant docker
SCRIPT

$script2 = <<-'SCRIPT'
#!/bin/bash
if [ -d "/home/vagrant/antani" ]; then
    cd /home/vagrant/antani
    docker-compose down
    cd ..
    rm -rf /home/vagrant/antani
    docker volume prune -f
fi
rm -rf /home/vagrant/geonode-project
SCRIPT

$script3 = <<-'SCRIPT'
sudo reboot
SCRIPT


Vagrant.configure(2) do |config|
    boxes = {
            'geonode-compose' => 'ubuntu/focal64'
    }
    machine_id = 0


    boxes.each do | key, value |
        config.vm.network "forwarded_port", guest: 80, host: 8888
        config.vm.provider "virtualbox" do |v|
            v.memory = 8132
            v.cpus = 4
            v.name = "geonode-vm"
        end
        config.vm.box = "#{value}"
        machine_id = machine_id + 1
        config.vm.define "#{key}" do |node|
            node.vm.hostname = "#{key}"
        config.vm.provision "shell", inline: $script1, privileged: false
        config.vm.provision "shell", inline: $script2, run: 'always', privileged: true
        config.vm.provision "file", source: $geonode_source_path, destination: "$HOME/geonode-project", run: 'always'
        config.vm.provision :shell, path: "antani-vagrant-compose.sh", run: 'always', privileged: false
        config.vm.provision "shell", inline: $script3, run: 'always', privileged: false
        end
    end
end