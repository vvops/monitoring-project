Vagrant.configure("2") do |mashin|
    mashin.vm.define "server" do |config|
        config.vm.box = "generic/ubuntu2004"
        config.vm.network "private_network", ip: "10.0.2.15", virtualbox__intnet: true
        config.vm.network "forwarded_port", id: "n", host: 9090, guest: 9090
        config.vm.network "forwarded_port", id: "s", host: 9100, guest: 9100
        config.vm.network "forwarded_port", id: "f", host: 3000, guest: 3000
        config.vm.hostname = "TEST"
        
        config.vm.provider "virtualbox" do |serv|
            serv.name = "test-work"
            serv.memory = 1500
            serv.cpus = 2
        end

        config.vm.provision "shell" do |i|
            run = 'ansible localhost -b -m'
            i.inline = <<-SHELL        
        
            echo ">>>>>>>>>>>>>update cache"
            apt-get update
            echo ">>>>>>>>>>>>> ansible install"
            apt-get install -y ansible
            echo ">>>>>>>>>>>>> ansible ping"
            ansible localhost -m ping
            
            echo ">>>>>>>>>>>>> ansible-ADhoc-install docker"
            #{run} file -a "path=/etc/apt/keyrings state=directory"
            #{run} apt_key -a "url=https://download.docker.com/linux/ubuntu/gpg state=present keyring=/etc/apt/keyrings/docker.gpg"
            #{run} apt_repository -a "repo='deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu focal stable' state=present"
            #{run} apt -a "name='docker-ce, docker-ce-cli, python3-docker, containerd.io, docker-compose-plugin'"
            #{run} command -a "ansible-galaxy collection install community.docker"
            echo ">>>>>>>>>>>>> docker status"
            #{run} systemd -a "name=docker state=restarted"
            #{run} command -a "systemctl status docker"
            apt-get update
            echo ">>>>>>>>>>>>> ansible-ADhoc-install docker-compose"        
            #{run} get_url -a "url=https://github.com/docker/compose/releases/download/v2.6.0/docker-compose-linux-x86_64 dest=/usr/local/bin/docker-compose mode=0755"
            #{run} apt -a "name=docker-compose"
            echo ">>>>>>>>>>>>> add user in group docker"
            #{run} user -a "name=$USER group=docker append=true"
            
            echo ">>>>>>>>>>>> install node_exporter"
            #{run} apt -a "name=prometheus-node-exporter"
            SHELL
        end
        
        config.vm.provision "shell" do |s|
            go = 'ansible localhost -b -m'
            add = 'ansible localhost -b -m command -a'
            s.inline = <<-SHELL
            
            echo ">>>>>>>>>>>>> create volume"
            #{go} git -a "repo=https://github.com/vvops/test-work.git dest=/tmp/git clone=true update=true"
            #{go} docker_volume -a "name=prometheus state=present"
            #{go} apt -a "update-cache=true"
            
            echo ">>>>>>>>>>>>> up docker-compose"
            #{go} file -a "path=/usr/local/project state=directory"
            #{add} "cp -a /tmp/git/. /usr/local/project"
            #{go} community.docker.docker_compose -a "project_src=/usr/local/project files=docker-compose.yml" 
            #{add} "cp /usr/local/project/prometheus.yml /var/lib/docker/volumes/prometheus/_data"
            #{go} community.docker.docker_compose -a "project_src=/usr/local/project restarted=true"
            SHELL
        end
    end
end
