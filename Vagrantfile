# -*- mode: ruby -*-
# vi: set ft=ruby :

cfg = {
  :vm_box => "generic/ubuntu1804",  #"generic/rhel7", "centos/7",
  :vm_count => 0,
  :vm_prefix => "ubu",
  :vm_cpus => 2,
  :vm_memory => 2048,
  :ssh_priv => "/path/to/ssh/private/key", 
  :ssh_public => "/path/to/ssh/public/key",
  :network_switch => "switchname"
}
  
boxes = [
  {
    :name => "Kubernetes Master", 
    :hostname => "kmaster",
    :memory => 2048,
    :cpus => 2, 
  },
  {
    :name => "Kubernetes Node 1", 
    :hostname => "knode1",
    :memory => 2048,
    :cpus => 2
  }, 
  {
    :name => "Kubernetes Node 2", 
    :hostname => "knode2",
    :memory => 2048,
    :cpus => 2, 
  }
]
  
if cfg[:vm_count] > 0
  boxes = []
  for i in 1..cfg[:vm_count] do
    m = {
      :name => cfg[:vm_prefix] + i.to_s, 
      :hostname => cfg[:vm_prefix] + i.to_s,
      :memory => cfg[:vm_memory],
      :cpus => cfg[:cpus], 
    } 
    boxes << m
  end
end
  
Vagrant.configure(2) do |config|
  config.vm.box = "#{cfg[:vm_box]}"
  config.vm.network "public_network", bridge: "#{cfg[:network_switch]}"

  config.vm.provision "file", source: "#{cfg[:ssh_priv]}", destination: "/home/vagrant/.ssh/id_rsa"
  public_key = File.read("#{cfg[:ssh_public]}") 
  config.vm.provision :shell, :inline =>"
          echo 'Copying SSH Keys to the VM'
          mkdir -p /home/vagrant/.ssh
          chmod 700 /home/vagrant/.ssh
          echo '#{public_key}' >> /home/vagrant/.ssh/authorized_keys
          chmod -R 600 /home/vagrant/.ssh/authorized_keys
          echo 'Host *' >> /home/vagrant/.ssh/config
          echo 'StrictHostKeyChecking no' >> /home/vagrant/.ssh/config
          echo 'UserKnownHostsFile /dev/null' >> /home/vagrant/.ssh/config
          chmod -R 600 /home/vagrant/.ssh/config
          echo 'Connection information:'
          ip=$(hostname -I | awk '{print $1}')
          echo ip: $ip
          echo hostname: $HOSTNAME
          ", privileged: false
  
  boxes.each do |machine|
    hostname = cfg[:vm_prefix] + "-" + machine[:hostname]
    config.vm.define "#{hostname}" do |m|
      m.vm.hostname = hostname
      m.vm.provider "hyperv" do |v|
        v.vmname = cfg[:vm_prefix] + ": " + machine[:name]
        v.memory = machine[:memory]
        v.cpus = machine[:cpus]
        v.linked_clone = true
        v.mac = "525400" + Array.new(6){[*"A".."F", *"0".."9"].sample}.join
      end;
    end
  end
end