VAGRANTFILE_API_VERSION = "2"

cluster = {
  "discovery1" => { :ip => "192.168.17.10", :cpus => 2, :mem => 2048 },
  "discovery2" => { :ip => "192.168.17.20", :cpus => 2, :mem => 2048 },
  "discovery3" => { :ip => "192.168.17.30", :cpus => 2, :mem => 2048 },
  "worker1"    => { :ip => "192.168.17.100", :cpus => 4, :mem => 6144 },
  "worker2"    => { :ip => "192.168.17.110", :cpus => 4, :mem => 6144 },
  "worker3"    => { :ip => "192.168.17.120", :cpus => 4, :mem => 6144 }
}

groups = {
  "zookeeper" => ["discovery1", "discovery2", "discovery3"],
  "mesos-master" => ["discovery1", "discovery2", "discovery3"],
  "mesos-agent" => ["worker1", "worker2", "worker3"],
  "mesos:children" => ["mesos-master", "mesos-agent"]
}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  cluster.each_with_index do |(hostname, info), index|

    config.vm.define hostname do |cfg|
      cfg.vm.hostname = hostname
      cfg.vm.box = "debian/contrib-stretch64"
  
      cfg.vm.network :private_network, ip: info[:ip]
      
      cfg.vm.provider :virtualbox do |vb|
        vb.linked_clone = true
        vb.name = hostname
        vb.cpus = info[:cpus]
        vb.memory = info[:mem]
        vb.customize ["modifyvm", :id, "--ostype", "Debian_64"]
        vb.customize ["modifyvm", :id, "--pagefusion", "on"]
        vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
      end # end provider

    end # end config
    
  end # end cluster

  config.vm.provision :shell, inline: "sed -i \"/^127.0.0.1\\t$(hostname)/d\" /etc/hosts"

  config.vm.provision :ansible do |ansible|
    ansible.galaxy_role_file = 'requirements.yml'
    ansible.galaxy_roles_path = 'provisioning/roles-galaxy'
    ansible.groups = groups   
    ansible.playbook = "provisioning/playbook.yml"
  end # end provision

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true

  config.vbguest.auto_update = false

end
